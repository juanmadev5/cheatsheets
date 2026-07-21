# ASP.NET Core — Persistencia

---

## 🏗️ Entity Framework Core — entidades

```csharp
// BaseEntity — campos comunes heredados por todas las entidades
public abstract class BaseEntity
{
    public Guid Id { get; set; } = Guid.NewGuid();
    public DateTime CreatedAt { get; set; }
    public DateTime UpdatedAt { get; set; }
    public string? CreatedBy { get; set; }
    public string? UpdatedBy { get; set; }

    [ConcurrencyCheck]
    public uint Version { get; set; }      // Optimistic locking
}

// Entidad completa
public class Product : BaseEntity
{
    public required string Sku { get; set; }
    public required string Name { get; set; }
    public string? Description { get; set; }
    public decimal Price { get; set; }
    public int Stock { get; set; } = 0;
    public bool Active { get; set; } = true;
    public ProductStatus Status { get; set; } = ProductStatus.Draft;

    // Colección de tipos simples
    public List<string> Tags { get; set; } = [];

    // Diccionario de atributos (mapeado como JSON)
    public Dictionary<string, string> Attributes { get; set; } = new();

    // Relación muchos-a-uno — FK en esta tabla
    public Guid CategoryId { get; set; }
    public Category Category { get; set; } = null!;

    // Relación uno-a-muchos — colección
    public List<ProductImage> Images { get; set; } = [];

    // Columna JSON (PostgreSQL jsonb)
    public Dictionary<string, object>? Metadata { get; set; }

    // Métodos de gestión de relaciones
    public void AddImage(ProductImage image)
    {
        Images.Add(image);
        image.ProductId = Id;
    }
}

public enum ProductStatus { Draft, Active, Inactive, Discontinued }

// Fluent API — configuración explícita (recomendado sobre Data Annotations)
public class ProductConfiguration : IEntityTypeConfiguration<Product>
{
    public void Configure(EntityTypeBuilder<Product> builder)
    {
        builder.ToTable("products");

        builder.HasKey(p => p.Id);

        builder.Property(p => p.Sku)
            .IsRequired()
            .HasMaxLength(50);
        builder.HasIndex(p => p.Sku).IsUnique();

        builder.Property(p => p.Name)
            .IsRequired()
            .HasMaxLength(200);

        builder.Property(p => p.Price)
            .HasPrecision(10, 2);

        builder.Property(p => p.Status)
            .HasConversion<string>()
            .HasMaxLength(30);

        // Colección de strings como columna PostgreSQL (text[])
        builder.Property(p => p.Tags)
            .HasColumnType("text[]");

        // Columna JSONB
        builder.Property(p => p.Metadata)
            .HasColumnType("jsonb");

        builder.HasOne(p => p.Category)
            .WithMany(c => c.Products)
            .HasForeignKey(p => p.CategoryId)
            .OnDelete(DeleteBehavior.Restrict);

        builder.HasMany(p => p.Images)
            .WithOne(i => i.Product)
            .HasForeignKey(i => i.ProductId)
            .OnDelete(DeleteBehavior.Cascade);

        builder.HasIndex(p => p.Active);
        builder.HasIndex(p => p.CategoryId);

        // Query filter global — soft delete o multi-tenant
        builder.HasQueryFilter(p => p.Active);
    }
}

// DbContext
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    public DbSet<Product> Products => Set<Product>();
    public DbSet<Category> Categories => Set<Category>();
    public DbSet<User> Users => Set<User>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.ApplyConfigurationsFromAssembly(typeof(AppDbContext).Assembly);
    }

    // Auditoría automática de CreatedAt/UpdatedAt al guardar
    public override async Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
    {
        foreach (var entry in ChangeTracker.Entries<BaseEntity>())
        {
            if (entry.State == EntityState.Added)
                entry.Entity.CreatedAt = DateTime.UtcNow;
            if (entry.State is EntityState.Added or EntityState.Modified)
                entry.Entity.UpdatedAt = DateTime.UtcNow;
        }
        return await base.SaveChangesAsync(cancellationToken);
    }
}
```

---

## 🔍 DbContext y consultas

```csharp
// Repositorio con LINQ — EF Core traduce las expresiones a SQL
public interface IProductRepository
{
    Task<Product?> FindByIdAsync(Guid id);
    Task<Product?> FindBySkuAsync(string sku);
    Task<bool> ExistsBySkuAsync(string sku);
    Task<List<Product>> FindByActiveAsync();
    Task<(List<Product> Items, long Total)> SearchAsync(
        string? name, string? category, decimal? minPrice, decimal? maxPrice,
        int page, int size);
    Task SaveAsync(Product product);
    Task DeleteAsync(Guid id);
}

public class ProductRepository : IProductRepository
{
    private readonly AppDbContext _context;

    public ProductRepository(AppDbContext context)
    {
        _context = context;
    }

    public async Task<Product?> FindByIdAsync(Guid id) =>
        await _context.Products
            .Include(p => p.Category)
            .FirstOrDefaultAsync(p => p.Id == id);

    public async Task<Product?> FindBySkuAsync(string sku) =>
        await _context.Products.FirstOrDefaultAsync(p => p.Sku == sku);

    public async Task<bool> ExistsBySkuAsync(string sku) =>
        await _context.Products.AnyAsync(p => p.Sku == sku);

    public async Task<List<Product>> FindByActiveAsync() =>
        await _context.Products.Where(p => p.Active).ToListAsync();

    // Búsqueda dinámica combinando filtros opcionales — equivalente a Specification
    public async Task<(List<Product> Items, long Total)> SearchAsync(
        string? name, string? category, decimal? minPrice, decimal? maxPrice,
        int page, int size)
    {
        var query = _context.Products.Where(p => p.Active).AsQueryable();

        if (!string.IsNullOrEmpty(name))
            query = query.Where(p => EF.Functions.ILike(p.Name, $"%{name}%"));

        if (!string.IsNullOrEmpty(category))
            query = query.Where(p => p.Category.Name == category);

        if (minPrice.HasValue)
            query = query.Where(p => p.Price >= minPrice.Value);

        if (maxPrice.HasValue)
            query = query.Where(p => p.Price <= maxPrice.Value);

        var total = await query.LongCountAsync();
        var items = await query
            .OrderByDescending(p => p.CreatedAt)
            .Skip(page * size)
            .Take(size)
            .ToListAsync();

        return (items, total);
    }

    // Proyección — solo traer campos necesarios (evita cargar la entidad completa)
    public async Task<List<ProductSummary>> FindSummariesAsync() =>
        await _context.Products
            .Select(p => new ProductSummary
            {
                Id = p.Id,
                Name = p.Name,
                Price = p.Price,
                CategoryName = p.Category.Name
            })
            .ToListAsync();

    // Query con SQL nativo (cuando LINQ no alcanza)
    public async Task<List<Product>> FindActiveNativeAsync(int limit) =>
        await _context.Products
            .FromSqlInterpolated($"""
                SELECT p.* FROM products p
                WHERE p.active = true
                ORDER BY p.created_at DESC
                LIMIT {limit}
                """)
            .ToListAsync();

    // Bulk update — actualización masiva sin cargar entidades (EF Core 7+)
    public async Task<int> DecrementStockAsync(Guid id, int quantity) =>
        await _context.Products
            .Where(p => p.Id == id && p.Stock >= quantity)
            .ExecuteUpdateAsync(setters => setters
                .SetProperty(p => p.Stock, p => p.Stock - quantity));

    public async Task<int> DeactivateByIdsAsync(List<Guid> ids) =>
        await _context.Products
            .Where(p => ids.Contains(p.Id))
            .ExecuteUpdateAsync(setters => setters.SetProperty(p => p.Active, false));

    public async Task SaveAsync(Product product)
    {
        if (await _context.Products.AnyAsync(p => p.Id == product.Id))
            _context.Products.Update(product);
        else
            _context.Products.Add(product);
        await _context.SaveChangesAsync();
    }

    public async Task DeleteAsync(Guid id)
    {
        await _context.Products
            .Where(p => p.Id == id)
            .ExecuteDeleteAsync();
    }
}

public class ProductSummary
{
    public Guid Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public string CategoryName { get; set; } = string.Empty;
}
```

---

## 📄 Paginación y ordenamiento

```csharp
// Parámetros de paginación desde query string
public class PageRequest
{
    public int Page { get; set; } = 0;
    public int Size { get; set; } = 20;
    public string SortBy { get; set; } = "CreatedAt";
    public string SortDir { get; set; } = "desc";
}

// Extension method reutilizable para ordenar dinámicamente
public static class QueryableExtensions
{
    public static IQueryable<T> ApplySort<T>(
        this IQueryable<T> query, string sortBy, string sortDir)
    {
        var param = Expression.Parameter(typeof(T), "x");
        var property = Expression.PropertyOrField(param, sortBy);
        var lambda = Expression.Lambda(property, param);

        var methodName = sortDir.Equals("desc", StringComparison.OrdinalIgnoreCase)
            ? "OrderByDescending" : "OrderBy";

        var method = typeof(Queryable).GetMethods()
            .First(m => m.Name == methodName && m.GetParameters().Length == 2)
            .MakeGenericMethod(typeof(T), property.Type);

        return (IQueryable<T>)method.Invoke(null, [query, lambda])!;
    }
}

// Uso en el Service
public async Task<PageResponse<ProductResponse>> FindAllAsync(ProductQueryParams query)
{
    var size = Math.Min(query.Size, 100);   // Límite máximo de tamaño de página

    var baseQuery = _context.Products.Where(p => p.Active)
        .ApplySort(query.SortBy, query.SortDir);

    var total = await baseQuery.LongCountAsync();
    var items = await baseQuery
        .Skip(query.Page * size)
        .Take(size)
        .Select(p => p.Adapt<ProductResponse>())
        .ToListAsync();

    return PageResponse<ProductResponse>.From(items, query.Page, size, total);
}
```

---

## 🔄 Transacciones

```csharp
// Transacción explícita con IDbContextTransaction
public class OrderService
{
    private readonly AppDbContext _context;

    public async Task<OrderResponse> CreateOrderAsync(CreateOrderRequest request)
    {
        await using var transaction = await _context.Database.BeginTransactionAsync();
        try
        {
            var order = new Order { /* ... */ };
            _context.Orders.Add(order);
            await _context.SaveChangesAsync();

            await _context.Products
                .Where(p => p.Id == request.ProductId)
                .ExecuteUpdateAsync(s => s.SetProperty(p => p.Stock, p => p.Stock - request.Quantity));

            await transaction.CommitAsync();
            return order.Adapt<OrderResponse>();
        }
        catch
        {
            await transaction.RollbackAsync();
            throw;
        }
    }

    // ExecutionStrategy — recomendado cuando hay reintentos automáticos habilitados
    // (EnableRetryOnFailure), porque una transacción manual debe ejecutarse dentro
    // de la estrategia de resiliencia configurada en el DbContext
    public async Task CreateOrderWithStrategyAsync(CreateOrderRequest request)
    {
        var strategy = _context.Database.CreateExecutionStrategy();
        await strategy.ExecuteAsync(async () =>
        {
            await using var transaction = await _context.Database.BeginTransactionAsync();
            // ... operaciones ...
            await transaction.CommitAsync();
        });
    }

    // Aislamiento personalizado
    public async Task CriticalOperationAsync()
    {
        await using var transaction = await _context.Database
            .BeginTransactionAsync(IsolationLevel.Serializable);
        // ...
        await transaction.CommitAsync();
    }
}
// Niveles de aislamiento disponibles:
// ReadUncommitted, ReadCommitted (default PostgreSQL), RepeatableRead, Serializable
```

---

## 🗃️ Migraciones — EF Core

```bash
# Instalar herramienta CLI (una sola vez, global)
dotnet tool install --global dotnet-ef

# Crear una migración
dotnet ef migrations add CreateProductsTable

# Aplicar migraciones pendientes a la base de datos
dotnet ef database update

# Revertir a una migración específica
dotnet ef database update PreviousMigrationName

# Generar script SQL sin aplicar
dotnet ef migrations script

# Eliminar la última migración (si no fue aplicada)
dotnet ef migrations remove

# Listar migraciones
dotnet ef migrations list
```

```csharp
// Migración generada automáticamente (Migrations/xxxx_CreateProductsTable.cs)
public partial class CreateProductsTable : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateTable(
            name: "products",
            columns: table => new
            {
                id = table.Column<Guid>(nullable: false),
                sku = table.Column<string>(maxLength: 50, nullable: false),
                name = table.Column<string>(maxLength: 200, nullable: false),
                price = table.Column<decimal>(precision: 10, scale: 2, nullable: false),
                stock = table.Column<int>(nullable: false, defaultValue: 0),
                active = table.Column<bool>(nullable: false, defaultValue: true),
                created_at = table.Column<DateTime>(nullable: false)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_products", x => x.id);
            });

        migrationBuilder.CreateIndex(
            name: "IX_products_sku", table: "products", column: "sku", unique: true);
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropTable(name: "products");
    }
}

// Aplicar migraciones automáticamente al iniciar (útil en desarrollo, no en producción)
using (var scope = app.Services.CreateScope())
{
    var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
    if (app.Environment.IsDevelopment())
    {
        await db.Database.MigrateAsync();
    }
}
```

