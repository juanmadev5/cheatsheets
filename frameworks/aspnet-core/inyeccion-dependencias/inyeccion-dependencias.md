# ASP.NET Core — Inyección de dependencias

---

## 💉 Inyección de dependencias

```csharp
// Ciclos de vida de servicios
builder.Services.AddSingleton<ICacheService, CacheService>();   // Una instancia para toda la app
builder.Services.AddScoped<IProductService, ProductService>();  // Una instancia por request HTTP
builder.Services.AddTransient<IEmailBuilder, EmailBuilder>();   // Nueva instancia cada vez

// Inyección por constructor — estándar en ASP.NET Core
public class ProductService : IProductService
{
    private readonly IProductRepository _productRepository;
    private readonly ILogger<ProductService> _logger;

    // El framework resuelve automáticamente las dependencias registradas
    public ProductService(IProductRepository productRepository, ILogger<ProductService> logger)
    {
        _productRepository = productRepository;
        _logger = logger;
    }
}

// Registrar múltiples implementaciones — resolver con keyed services (.NET 8+)
builder.Services.AddKeyedScoped<INotificationService, EmailNotificationService>("email");
builder.Services.AddKeyedScoped<INotificationService, SmsNotificationService>("sms");

public class OrderService
{
    private readonly INotificationService _emailNotifier;
    private readonly INotificationService _smsNotifier;

    public OrderService(
        [FromKeyedServices("email")] INotificationService emailNotifier,
        [FromKeyedServices("sms")] INotificationService smsNotifier)
    {
        _emailNotifier = emailNotifier;
        _smsNotifier = smsNotifier;
    }
}

// Factory pattern para resolución condicional
builder.Services.AddScoped<Func<string, INotificationService>>(sp => key => key switch
{
    "email" => sp.GetRequiredKeyedService<INotificationService>("email"),
    "sms" => sp.GetRequiredKeyedService<INotificationService>("sms"),
    _ => throw new ArgumentException("Notificador desconocido")
});

// TryAdd — registrar solo si no existe otra implementación
builder.Services.TryAddScoped<ICacheService, MemoryCacheService>();

// Extension methods para organizar el registro de servicios por módulo
public static class ServiceCollectionExtensions
{
    public static IServiceCollection AddApplicationServices(this IServiceCollection services)
    {
        services.AddScoped<IProductService, ProductService>();
        services.AddScoped<IProductRepository, ProductRepository>();
        services.AddScoped<IAuthService, AuthService>();
        return services;
    }
}

// En Program.cs: builder.Services.AddApplicationServices();
```

---

## ⚙️ Configuración — Options pattern

```csharp
// Clase de opciones tipada
public class JwtOptions
{
    public const string SectionName = "Jwt";

    [Required]
    public string Secret { get; set; } = string.Empty;

    public string Issuer { get; set; } = string.Empty;
    public string Audience { get; set; } = string.Empty;

    [Range(1, 1440)]
    public int ExpirationMinutes { get; set; } = 60;

    public int RefreshExpirationDays { get; set; } = 7;
}

// Registro con validación al arrancar la aplicación
builder.Services
    .AddOptions<JwtOptions>()
    .Bind(builder.Configuration.GetSection(JwtOptions.SectionName))
    .ValidateDataAnnotations()
    .ValidateOnStart();

// Uso con IOptions (valor fijo, resuelto una vez)
public class JwtService
{
    private readonly JwtOptions _options;

    public JwtService(IOptions<JwtOptions> options)
    {
        _options = options.Value;
    }
}

// IOptionsSnapshot — recarga por cada request (útil en configuración recargable)
public class ProductController : ControllerBase
{
    public ProductController(IOptionsSnapshot<UploadOptions> uploadOptions) { }
}

// IOptionsMonitor — notifica cambios en tiempo real (para servicios Singleton)
public class FileUploadService
{
    public FileUploadService(IOptionsMonitor<UploadOptions> monitor)
    {
        monitor.OnChange(options => Console.WriteLine("Configuración actualizada"));
    }
}
```

