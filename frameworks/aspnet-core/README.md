# ASP.NET Core — Cheatsheet (.NET 10 + C# 14)

Cheatsheet de ASP.NET Core organizado por temas. Cada carpeta contiene el/los `.md` de su categoría.

---

## 📑 Índice

### Base
- [Proyecto (csproj), Program.cs, appsettings.json y comandos dotnet CLI](base/base.md)

### Web / API REST
- [Minimal APIs, Controllers, Results, manejo de excepciones y validación](web-api/web-api.md)

### DTOs y Mapeo
- [DTOs con record y Mapster/AutoMapper](dtos-mapeo/dtos-mapeo.md)

### Inyección de dependencias
- [Servicios, ciclos de vida y Options pattern](inyeccion-dependencias/inyeccion-dependencias.md)

### Persistencia
- [Entity Framework Core — entidades, DbContext, paginación, transacciones y migraciones](persistencia/persistencia.md)

### Seguridad
- [Autenticación JWT Bearer, ASP.NET Core Identity y autorización](seguridad/seguridad.md)

### Configuración avanzada
- [Options pattern avanzado, entornos y variables por ambiente](configuracion-avanzada/configuracion-avanzada.md)

### Features
- [Caché, tareas en segundo plano, Scheduling, MediatR, Mail y Health Checks](features/features.md)

### Testing
- [xUnit, WebApplicationFactory, Moq, FluentAssertions y Testcontainers](testing/testing.md)

### DevOps
- [Docker — Dockerfile y docker-compose.yml](devops/devops.md)

---

## 💡 Buenas prácticas y patrones

### Arquitectura por capas

```
Endpoint/Controller → recibe HTTP, delega al Service, devuelve Results/IActionResult
Service             → lógica de negocio, orquesta, usa Repository
Repository          → acceso a datos, sin lógica de negocio
Entity              → modelo de persistencia, solo EF Core
DTO                 → transferencia de datos entre capas (record)
Mapper              → convierte entre Entity y DTO (Mapster / AutoMapper)
Exception           → excepciones de dominio bien tipadas
```

### Reglas clave

```csharp
// ✅ Usar interfaces para Services — facilita testing y desacoplamiento
public interface IProductService
{
    Task<ProductResponse> FindByIdAsync(Guid id);
    Task<ProductResponse> CreateAsync(CreateProductRequest request);
}

// ✅ Nunca exponer entidades EF Core en los endpoints — siempre usar DTOs
// ❌ return await _context.Products.ToListAsync();
// ✅ return await _context.Products.Select(p => p.Adapt<ProductResponse>()).ToListAsync();

// ✅ Inicializar colecciones en las entidades para evitar NullReferenceException
public List<ProductImage> Images { get; set; } = [];
public List<string> Tags { get; set; } = [];

// ✅ Usar AsNoTracking() en consultas de solo lectura — mejora el rendimiento
var products = await _context.Products.AsNoTracking().ToListAsync();

// ✅ Evitar N+1 con Include() o proyecciones explícitas
var products = await _context.Products
    .Include(p => p.Category)
    .Include(p => p.Images)
    .ToListAsync();

// ✅ Preferir ExecuteUpdateAsync/ExecuteDeleteAsync para operaciones masivas
// (evita cargar entidades a memoria innecesariamente)
await _context.Products.Where(p => p.Stock == 0)
    .ExecuteUpdateAsync(s => s.SetProperty(p => p.Active, false));

// ✅ Validar en el Service, no solo en el endpoint
// El endpoint valida el formato (Data Annotations / FluentValidation)
// El Service valida las reglas de negocio

// ✅ Usar ILogger<T> — no crear loggers manualmente
// ❌ Console.WriteLine("Error: " + ex.Message);
// ✅ _logger.LogError(ex, "Error al crear producto");

// ✅ Nunca loguear datos sensibles
_logger.LogInformation("Login para usuario: {Email}", email);   // ✅
_logger.LogInformation("Password: {Password}", password);        // ❌ NUNCA

// ✅ Usar paginación siempre — nunca ToListAsync() sin límite en producción
// ❌ await _context.Products.ToListAsync()
// ✅ await _context.Products.Skip(page * size).Take(size).ToListAsync()

// ✅ Externalizar toda la configuración en appsettings.json / variables de entorno
// ❌ private string secret = "hardcoded-secret";
// ✅ options.Value.Secret (inyectado vía Options pattern)

// ✅ Configurar graceful shutdown (por defecto en Kestrel, ajustable con
// HostOptions.ShutdownTimeout)

// ✅ Versionar la API desde el inicio
// /api/v1/products
// /api/v2/products (breaking changes)

// ✅ Usar HTTP correcto:
// POST   → crear       → 201 Created
// GET    → leer        → 200 OK
// PUT    → reemplazar  → 200 OK
// PATCH  → actualizar  → 200 OK
// DELETE → eliminar    → 204 No Content
// GET inexistente      → 404 Not Found
// POST duplicado       → 409 Conflict
// POST inválido        → 400 Bad Request
```
