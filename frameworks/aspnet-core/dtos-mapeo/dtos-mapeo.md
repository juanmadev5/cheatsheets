# ASP.NET Core — DTOs y Mapeo

---

## 📋 DTOs con record

```csharp
// record — inmutable, ideal para DTOs de respuesta
public record ProductResponse(
    Guid Id,
    string Name,
    string? Description,
    decimal Price,
    int Stock,
    string Category,
    string Sku,
    bool Active,
    DateTime CreatedAt)
{
    // Métodos de utilidad
    public string FormattedPrice => $"${Price:F2}";
    public bool IsInStock => Stock > 0;
}

// Request DTO
public class CreateProductRequest
{
    public required string Name { get; set; }
    public decimal Price { get; set; }
    public int Stock { get; set; }
    public required string Category { get; set; }
    public List<string> Tags { get; set; } = [];
}

// DTOs de autenticación
public record LoginRequest(string Email, string Password);

public record AuthResponse(
    string AccessToken,
    string RefreshToken,
    long ExpiresIn,
    UserResponse User);

public record UserResponse(
    Guid Id,
    string Name,
    string Email,
    string Role,
    DateTime CreatedAt);

// DTO con datos anidados
public record OrderResponse(
    Guid Id,
    string Status,
    decimal Total,
    UserResponse Customer,
    List<OrderItemResponse> Items,
    AddressResponse ShippingAddress,
    DateTime CreatedAt);

public record OrderItemResponse(
    Guid Id,
    ProductResponse Product,
    int Quantity,
    decimal UnitPrice,
    decimal Subtotal);
```

---

## 🗺️ Mapster / AutoMapper — mapeo automático

```csharp
// Mapster — configuración de mapeo (alternativa liviana a AutoMapper)
public class ProductMappingConfig : IRegister
{
    public void Register(TypeAdapterConfig config)
    {
        // Mapeo directo (campos con mismo nombre — automático sin configurar)
        config.NewConfig<Product, ProductResponse>();

        // Mapeo con nombres diferentes / propiedades anidadas
        config.NewConfig<Product, ProductSummaryResponse>()
            .Map(dest => dest.ProductName, src => src.Name)
            .Map(dest => dest.CategoryName, src => src.Category.Name);

        // Ignorar campos al mapear el request a la entidad
        config.NewConfig<CreateProductRequest, Product>()
            .Ignore(dest => dest.Id)
            .Ignore(dest => dest.CreatedAt)
            .Ignore(dest => dest.UpdatedAt);
    }
}

// Registro en Program.cs
builder.Services.AddMapster();
TypeAdapterConfig.GlobalSettings.Scan(typeof(Program).Assembly);

// Uso en el Service
public class ProductService : IProductService
{
    private readonly IProductRepository _productRepository;

    public async Task<ProductResponse> FindByIdAsync(Guid id)
    {
        var product = await _productRepository.FindByIdAsync(id)
            ?? throw new ResourceNotFoundException("Product", id);
        return product.Adapt<ProductResponse>();
    }

    public async Task<ProductResponse> CreateAsync(CreateProductRequest request)
    {
        var product = request.Adapt<Product>();
        await _productRepository.SaveAsync(product);
        return product.Adapt<ProductResponse>();
    }

    // Actualizar entidad existente in-place (evita new + set uno por uno)
    public async Task<ProductResponse> UpdateAsync(Guid id, UpdateProductRequest request)
    {
        var product = await _productRepository.FindByIdAsync(id)
            ?? throw new ResourceNotFoundException("Product", id);
        request.Adapt(product);          // Mapea sobre la instancia existente
        await _productRepository.SaveAsync(product);
        return product.Adapt<ProductResponse>();
    }
}

// AutoMapper — alternativa más establecida
// Licencia dual RPL-1.5 / comercial desde la v15 (jul 2025) — gratuita para individuos y
// empresas con facturación anual menor a USD 5M (ver automapper.io)
public class ProductProfile : Profile
{
    public ProductProfile()
    {
        CreateMap<Product, ProductResponse>();
        CreateMap<CreateProductRequest, Product>()
            .ForMember(dest => dest.Id, opt => opt.Ignore());
    }
}

// Registro: builder.Services.AddAutoMapper(typeof(Program));
// Uso: _mapper.Map<ProductResponse>(product);
```

