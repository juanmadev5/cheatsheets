# ASP.NET Core — Testing

---

## 🧪 Testing

```csharp
// xUnit v3 (paquete xunit.v3) + WebApplicationFactory — test de integración con Testcontainers
// El proyecto de tests requiere <OutputType>Exe</OutputType> en el csproj (v3 genera un
// ejecutable standalone en lugar de una librería)
public class ProductEndpointsTests : IClassFixture<WebApplicationFactory<Program>>, IAsyncLifetime
{
    private readonly WebApplicationFactory<Program> _factory;
    private readonly PostgreSqlContainer _postgres = new PostgreSqlBuilder()
        .WithImage("postgres:16-alpine")
        .WithDatabase("test_db")
        .Build();

    private HttpClient _client = null!;

    public ProductEndpointsTests(WebApplicationFactory<Program> factory)
    {
        _factory = factory;
    }

    // IAsyncLifetime usa ValueTask en xUnit v3 (antes Task en v2)
    public async ValueTask InitializeAsync()
    {
        await _postgres.StartAsync();

        var customizedFactory = _factory.WithWebHostBuilder(builder =>
        {
            builder.ConfigureServices(services =>
            {
                services.RemoveAll<DbContextOptions<AppDbContext>>();
                services.AddDbContext<AppDbContext>(options =>
                    options.UseNpgsql(_postgres.GetConnectionString()));
            });
        });

        _client = customizedFactory.CreateClient();

        using var scope = customizedFactory.Services.CreateScope();
        var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
        await db.Database.MigrateAsync();
    }

    public async ValueTask DisposeAsync() => await _postgres.DisposeAsync();

    [Fact]
    public async Task CreateProduct_ReturnsCreated()
    {
        var request = new CreateProductRequest
        {
            Name = "Laptop",
            Price = 999.99m,
            Stock = 10,
            Category = "electronica"
        };

        var response = await _client.PostAsJsonAsync("/api/v1/products", request);

        response.StatusCode.Should().Be(HttpStatusCode.Created);
        var body = await response.Content.ReadFromJsonAsync<ProductResponse>();
        body.Should().NotBeNull();
        body!.Name.Should().Be("Laptop");
    }

    [Fact]
    public async Task GetById_NonExistingId_ReturnsNotFound()
    {
        var response = await _client.GetAsync($"/api/v1/products/{Guid.NewGuid()}");
        response.StatusCode.Should().Be(HttpStatusCode.NotFound);
    }
}
```

---

## 🎯 Moq, FluentAssertions y Testcontainers

```csharp
// FluentAssertions 8+ requiere licencia comercial de pago (Xceed) — se usa la última
// versión 7.x, que sigue siendo Apache 2.0 gratuita
// Test unitario de Service con Moq
public class ProductServiceTests
{
    private readonly Mock<IProductRepository> _repositoryMock = new();
    private readonly Mock<IMediator> _mediatorMock = new();
    private readonly ProductService _service;

    public ProductServiceTests()
    {
        _service = new ProductService(_repositoryMock.Object, _mediatorMock.Object);
    }

    [Fact]
    public async Task FindByIdAsync_ExistingId_ReturnsProduct()
    {
        var productId = Guid.NewGuid();
        var product = new Product { Id = productId, Name = "Laptop", Sku = "PROD-0001" };

        _repositoryMock.Setup(r => r.FindByIdAsync(productId))
            .ReturnsAsync(product);

        var result = await _service.FindByIdAsync(productId);

        result.Name.Should().Be("Laptop");
        _repositoryMock.Verify(r => r.FindByIdAsync(productId), Times.Once);
    }

    [Fact]
    public async Task FindByIdAsync_NonExistingId_ThrowsException()
    {
        var productId = Guid.NewGuid();
        _repositoryMock.Setup(r => r.FindByIdAsync(productId))
            .ReturnsAsync((Product?)null);

        Func<Task> act = () => _service.FindByIdAsync(productId);

        await act.Should().ThrowAsync<ResourceNotFoundException>()
            .WithMessage($"*{productId}*");
    }

    [Theory]
    [InlineData(-1)]
    [InlineData(0)]
    public async Task CreateAsync_InvalidPrice_ThrowsException(decimal price)
    {
        var request = new CreateProductRequest { Name = "Test", Price = price, Category = "test" };

        Func<Task> act = () => _service.CreateAsync(request);

        await act.Should().ThrowAsync<BusinessException>();
    }
}

// Test de repositorio con EF Core InMemory (rápido, para casos simples)
public class ProductRepositoryTests
{
    private static AppDbContext CreateContext()
    {
        var options = new DbContextOptionsBuilder<AppDbContext>()
            .UseInMemoryDatabase(Guid.NewGuid().ToString())
            .Options;
        return new AppDbContext(options);
    }

    [Fact]
    public async Task ExistsBySkuAsync_ExistingSku_ReturnsTrue()
    {
        await using var context = CreateContext();
        context.Products.Add(new Product { Sku = "TEST-001", Name = "Test", Price = 50 });
        await context.SaveChangesAsync();

        var repository = new ProductRepository(context);

        var exists = await repository.ExistsBySkuAsync("TEST-001");

        exists.Should().BeTrue();
    }
}
```

