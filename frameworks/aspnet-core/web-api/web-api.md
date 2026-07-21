# ASP.NET Core — Web / API REST

---

## 🌐 Minimal APIs

```csharp
// Endpoints/ProductEndpoints.cs — extension method para organizar rutas
public static class ProductEndpoints
{
    public static void MapProductEndpoints(this WebApplication app)
    {
        var group = app.MapGroup("/api/v1/products")
            .WithTags("Products");

        // GET — listar con paginación
        group.MapGet("/", async (
                [AsParameters] ProductQueryParams query,
                IProductService productService) =>
            {
                var result = await productService.FindAllAsync(query);
                return Results.Ok(result);
            })
            .WithName("GetProducts")
            .Produces<PageResponse<ProductResponse>>(StatusCodes.Status200OK);

        // GET — obtener por ID
        group.MapGet("/{id:guid}", async (Guid id, IProductService productService) =>
            {
                var product = await productService.FindByIdAsync(id);
                return Results.Ok(product);
            })
            .WithName("GetProductById")
            .Produces<ProductResponse>(StatusCodes.Status200OK)
            .Produces(StatusCodes.Status404NotFound);

        // POST — crear
        group.MapPost("/", async (
                CreateProductRequest request,
                IProductService productService) =>
            {
                var product = await productService.CreateAsync(request);
                return Results.Created($"/api/v1/products/{product.Id}", product);
            })
            .RequireAuthorization("AdminOnly")
            .Produces<ProductResponse>(StatusCodes.Status201Created)
            .ProducesValidationProblem();

        // PUT — actualización completa
        group.MapPut("/{id:guid}", async (
                Guid id,
                UpdateProductRequest request,
                IProductService productService) =>
            {
                var product = await productService.UpdateAsync(id, request);
                return Results.Ok(product);
            })
            .RequireAuthorization("AdminOnly");

        // PATCH — actualización parcial
        group.MapPatch("/{id:guid}", async (
                Guid id,
                Dictionary<string, object> fields,
                IProductService productService) =>
            {
                var product = await productService.PatchAsync(id, fields);
                return Results.Ok(product);
            })
            .RequireAuthorization("AdminOnly");

        // DELETE
        group.MapDelete("/{id:guid}", async (Guid id, IProductService productService) =>
            {
                await productService.DeleteAsync(id);
                return Results.NoContent();
            })
            .RequireAuthorization("AdminOnly");

        // GET — búsqueda con filtros
        group.MapGet("/search", async (
                string? name,
                string? category,
                decimal? minPrice,
                decimal? maxPrice,
                IProductService productService) =>
            {
                var results = await productService.SearchAsync(name, category, minPrice, maxPrice);
                return Results.Ok(results);
            });

        // POST — subir archivo
        group.MapPost("/{id:guid}/image", async (
                Guid id,
                IFormFile file,
                IProductService productService) =>
            {
                var product = await productService.UploadImageAsync(id, file);
                return Results.Ok(product);
            })
            .DisableAntiforgery();

        // GET — descargar archivo
        group.MapGet("/{id:guid}/export", async (Guid id, IProductService productService) =>
            {
                var (stream, fileName) = await productService.ExportToCsvAsync(id);
                return Results.File(stream, "text/csv", fileName);
            });

        // GET — con header personalizado
        group.MapGet("/by-sku/{sku}", async (
                string sku,
                [FromHeader(Name = "X-Tenant-Id")] string? tenantId,
                IProductService productService) =>
            {
                var product = await productService.FindBySkuAsync(sku, tenantId);
                return Results.Ok(product);
            });
    }
}

// Query params tipados con [AsParameters]
public class ProductQueryParams
{
    public int Page { get; set; } = 0;
    public int Size { get; set; } = 20;
    public string SortBy { get; set; } = "CreatedAt";
    public string SortDir { get; set; } = "desc";
    public string? Search { get; set; }
}
```

---

## 🎛 Controllers — MVC clásico

```csharp
[ApiController]
[Route("api/v1/[controller]")]
[Produces("application/json")]
public class ProductController : ControllerBase
{
    private readonly IProductService _productService;

    public ProductController(IProductService productService)
    {
        _productService = productService;
    }

    // GET — listar con paginación
    [HttpGet]
    [ProducesResponseType(typeof(PageResponse<ProductResponse>), StatusCodes.Status200OK)]
    public async Task<IActionResult> FindAll([FromQuery] ProductQueryParams query)
    {
        var result = await _productService.FindAllAsync(query);
        return Ok(result);
    }

    // GET — obtener por ID
    [HttpGet("{id:guid}")]
    [ProducesResponseType(typeof(ProductResponse), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> FindById(Guid id)
    {
        var product = await _productService.FindByIdAsync(id);
        return Ok(product);
    }

    // POST — crear
    [HttpPost]
    [Authorize(Policy = "AdminOnly")]
    public async Task<IActionResult> Create([FromBody] CreateProductRequest request)
    {
        var product = await _productService.CreateAsync(request);
        return CreatedAtAction(nameof(FindById), new { id = product.Id }, product);
    }

    // PUT — actualización completa
    [HttpPut("{id:guid}")]
    [Authorize(Policy = "AdminOnly")]
    public async Task<IActionResult> Update(Guid id, [FromBody] UpdateProductRequest request)
    {
        var product = await _productService.UpdateAsync(id, request);
        return Ok(product);
    }

    // DELETE
    [HttpDelete("{id:guid}")]
    [Authorize(Policy = "AdminOnly")]
    public async Task<IActionResult> Delete(Guid id)
    {
        await _productService.DeleteAsync(id);
        return NoContent();
    }
}
```

---

## 📨 Results y respuestas estandarizadas

```csharp
// ApiResponse — wrapper genérico para todas las respuestas
public class ApiResponse<T>
{
    public bool Success { get; init; }
    public string? Message { get; init; }
    public T? Data { get; init; }
    public List<string>? Errors { get; init; }
    public DateTime Timestamp { get; init; } = DateTime.UtcNow;

    public static ApiResponse<T> Ok(T data) =>
        new() { Success = true, Data = data };

    public static ApiResponse<T> Ok(T data, string message) =>
        new() { Success = true, Data = data, Message = message };

    public static ApiResponse<T> Error(string message) =>
        new() { Success = false, Message = message };

    public static ApiResponse<T> Error(string message, List<string> errors) =>
        new() { Success = false, Message = message, Errors = errors };
}

// PageResponse — respuesta paginada estandarizada
public class PageResponse<T>
{
    public required List<T> Content { get; init; }
    public int Page { get; init; }
    public int Size { get; init; }
    public long TotalElements { get; init; }
    public int TotalPages { get; init; }
    public bool First { get; init; }
    public bool Last { get; init; }
    public bool Empty { get; init; }

    public static PageResponse<T> From(List<T> items, int page, int size, long totalElements)
    {
        var totalPages = (int)Math.Ceiling(totalElements / (double)size);
        return new PageResponse<T>
        {
            Content = items,
            Page = page,
            Size = size,
            TotalElements = totalElements,
            TotalPages = totalPages,
            First = page == 0,
            Last = page >= totalPages - 1,
            Empty = items.Count == 0
        };
    }
}

// Respuestas HTTP comunes con Results (Minimal APIs)
Results.Ok(body);                                  // 200
Results.Created(uri, body);                        // 201
Results.Accepted(uri, body);                        // 202
Results.NoContent();                                // 204
Results.BadRequest(error);                          // 400
Results.Forbid();                                   // 403
Results.NotFound();                                 // 404
Results.Conflict(error);                            // 409
Results.Problem(detail: "Error interno", statusCode: 500);  // 500

// Respuestas HTTP comunes con ControllerBase (MVC)
Ok(body);                          // 200
CreatedAtAction(nameof(FindById), new { id }, body); // 201
NoContent();                       // 204
BadRequest(error);                 // 400
Forbid();                          // 403
NotFound();                        // 404
Conflict(error);                   // 409
StatusCode(500, error);            // 500
```

---

## ⚠️ Manejo global de excepciones

```csharp
// Excepciones de dominio
public class ResourceNotFoundException : Exception
{
    public ResourceNotFoundException(string resource, object id)
        : base($"{resource} no encontrado con id: {id}") { }
}

public class BusinessException : Exception
{
    public string Code { get; }

    public BusinessException(string code, string message) : base(message)
    {
        Code = code;
    }
}

public class DuplicateResourceException : Exception
{
    public DuplicateResourceException(string message) : base(message) { }
}

// IExceptionHandler — patrón moderno (.NET 8+) para manejo global de excepciones
public class GlobalExceptionHandler : IExceptionHandler
{
    private readonly ILogger<GlobalExceptionHandler> _logger;

    public GlobalExceptionHandler(ILogger<GlobalExceptionHandler> logger)
    {
        _logger = logger;
    }

    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken)
    {
        var (statusCode, title, detail) = exception switch
        {
            ResourceNotFoundException ex => (StatusCodes.Status404NotFound, "Recurso no encontrado", ex.Message),
            BusinessException ex => (StatusCodes.Status400BadRequest, "Error de negocio", ex.Message),
            DuplicateResourceException ex => (StatusCodes.Status409Conflict, "Conflicto", ex.Message),
            FluentValidation.ValidationException ex => (StatusCodes.Status400BadRequest, "Error de validación", ex.Message),
            UnauthorizedAccessException => (StatusCodes.Status403Forbidden, "Acceso denegado", "No tiene permisos para realizar esta acción"),
            _ => (StatusCodes.Status500InternalServerError, "Error del servidor", "Error interno del servidor")
        };

        if (statusCode == StatusCodes.Status500InternalServerError)
        {
            _logger.LogError(exception, "Error inesperado en {Path}", httpContext.Request.Path);
        }
        else
        {
            _logger.LogWarning("{Title}: {Detail}", title, detail);
        }

        var problemDetails = new ProblemDetails
        {
            Status = statusCode,
            Title = title,
            Detail = detail,
            Instance = httpContext.Request.Path
        };

        if (exception is BusinessException businessEx)
        {
            problemDetails.Extensions["code"] = businessEx.Code;
        }

        httpContext.Response.StatusCode = statusCode;
        await httpContext.Response.WriteAsJsonAsync(problemDetails, cancellationToken);

        return true;
    }
}

// Registro en Program.cs
builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
builder.Services.AddProblemDetails();
// ...
app.UseExceptionHandler();

// Alternativa con middleware clásico (más control manual)
public class ExceptionMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ExceptionMiddleware> _logger;

    public ExceptionMiddleware(RequestDelegate next, ILogger<ExceptionMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error no manejado");
            context.Response.StatusCode = StatusCodes.Status500InternalServerError;
            context.Response.ContentType = "application/json";
            await context.Response.WriteAsJsonAsync(new { error = "Error interno del servidor" });
        }
    }
}
```

---

## ✅ Validación

```csharp
// Data Annotations — validación básica integrada
public class CreateProductRequest
{
    [Required(ErrorMessage = "El nombre es obligatorio")]
    [StringLength(100, MinimumLength = 2,
        ErrorMessage = "El nombre debe tener entre 2 y 100 caracteres")]
    public string Name { get; set; } = string.Empty;

    [StringLength(500, ErrorMessage = "La descripción no puede superar los 500 caracteres")]
    public string? Description { get; set; }

    [Required(ErrorMessage = "El precio es obligatorio")]
    [Range(0.01, 999999.99, ErrorMessage = "El precio debe estar entre 0.01 y 999999.99")]
    public decimal Price { get; set; }

    [Required(ErrorMessage = "El stock es obligatorio")]
    [Range(0, int.MaxValue, ErrorMessage = "El stock no puede ser negativo")]
    public int Stock { get; set; }

    [Required(ErrorMessage = "La categoría es obligatoria")]
    public string Category { get; set; } = string.Empty;

    [RegularExpression(@"^[A-Z]{2,}-\d{4,}$",
        ErrorMessage = "El SKU debe tener formato XX-NNNN (ej: PROD-0001)")]
    public string? Sku { get; set; }

    [EmailAddress(ErrorMessage = "El email del proveedor es inválido")]
    public string? SupplierEmail { get; set; }

    [Url(ErrorMessage = "La URL de imagen es inválida")]
    public string? ImageUrl { get; set; }
}

// FluentValidation — recomendado para reglas complejas y desacopladas del DTO
public class CreateProductRequestValidator : AbstractValidator<CreateProductRequest>
{
    private readonly IProductRepository _productRepository;

    public CreateProductRequestValidator(IProductRepository productRepository)
    {
        _productRepository = productRepository;

        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("El nombre es obligatorio")
            .Length(2, 100).WithMessage("El nombre debe tener entre 2 y 100 caracteres");

        RuleFor(x => x.Price)
            .GreaterThan(0).WithMessage("El precio debe ser mayor a 0")
            .LessThanOrEqualTo(999999.99m);

        RuleFor(x => x.Stock)
            .GreaterThanOrEqualTo(0).WithMessage("El stock no puede ser negativo");

        RuleFor(x => x.Sku)
            .Matches(@"^[A-Z]{2,}-\d{4,}$")
            .When(x => !string.IsNullOrEmpty(x.Sku))
            .MustAsync(async (sku, cancellation) =>
                !await _productRepository.ExistsBySkuAsync(sku!))
            .WithMessage("El SKU ya está en uso");

        RuleFor(x => x.SupplierEmail)
            .EmailAddress()
            .When(x => !string.IsNullOrEmpty(x.SupplierEmail));

        RuleForEach(x => x.Tags)
            .NotEmpty().MaximumLength(30);

        RuleFor(x => x.Tags)
            .Must(tags => tags == null || tags.Count <= 10)
            .WithMessage("Máximo 10 etiquetas");
    }
}

// Registro de validadores — la validación se invoca manualmente (recomendado por
// FluentValidation: el pipeline automático de FluentValidation.AspNetCore no es async
// y no soporta Minimal APIs, solo MVC/Razor Pages)
builder.Services.AddValidatorsFromAssemblyContaining<Program>();

// Validación manual en un endpoint Minimal API
app.MapPost("/api/v1/products", async (
        CreateProductRequest request,
        IValidator<CreateProductRequest> validator,
        IProductService productService) =>
    {
        var validationResult = await validator.ValidateAsync(request);
        if (!validationResult.IsValid)
            return Results.ValidationProblem(validationResult.ToDictionary());

        var product = await productService.CreateAsync(request);
        return Results.Created($"/api/v1/products/{product.Id}", product);
    });

// Validación manual en el Service (cuando la regla depende de acceso a datos)
public class ProductService
{
    private readonly IValidator<CreateProductRequest> _validator;

    public async Task<ProductResponse> CreateAsync(CreateProductRequest request)
    {
        var validationResult = await _validator.ValidateAsync(request);
        if (!validationResult.IsValid)
        {
            var errors = validationResult.Errors.Select(e => $"{e.PropertyName}: {e.ErrorMessage}").ToList();
            throw new BusinessException("VALIDATION_ERROR",
                "Errores de validación: " + string.Join(", ", errors));
        }
        // ...
    }
}
```

