# ASP.NET Core — Configuración avanzada

---

## ⚙️ Options pattern avanzado

```csharp
// Clase de opciones con secciones anidadas
public class AppOptions
{
    public const string SectionName = "App";
    public UploadOptions Upload { get; set; } = new();
    public CorsOptions Cors { get; set; } = new();
}

public class UploadOptions
{
    public string Directory { get; set; } = "/tmp/uploads";
    public long MaxSizeBytes { get; set; } = 10_485_760;
    public List<string> AllowedTypes { get; set; } = [];
}

public class CorsOptions
{
    public List<string> AllowedOrigins { get; set; } = [];
    public List<string> AllowedMethods { get; set; } = ["GET", "POST", "PUT", "DELETE"];
}

// PostConfigure — modificar opciones después de vincularlas
builder.Services.PostConfigure<UploadOptions>(options =>
{
    if (!Directory.Exists(options.Directory))
        Directory.CreateDirectory(options.Directory);
});

// Named options — múltiples configuraciones del mismo tipo
builder.Services.Configure<UploadOptions>("Images", builder.Configuration.GetSection("Uploads:Images"));
builder.Services.Configure<UploadOptions>("Documents", builder.Configuration.GetSection("Uploads:Documents"));
```

---

## 🎭 Entornos

```csharp
// Comportamiento condicional según el entorno activo
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
    app.MapOpenApi();
    app.MapScalarApiReference();
}
else if (app.Environment.IsStaging())
{
    app.MapOpenApi();   // Habilitar el documento OpenAPI también en staging
}
else
{
    app.UseExceptionHandler();
    app.UseHsts();
}

// Registro condicional de servicios
if (builder.Environment.IsDevelopment())
{
    builder.Services.AddScoped<INotificationService, FakeNotificationService>();
}
else
{
    builder.Services.AddScoped<INotificationService, EmailNotificationService>();
}

// Verificar un entorno personalizado
if (app.Environment.IsEnvironment("QA"))
{
    // ...
}
```

---

## 🌍 Variables por ambiente

```bash
# Activar entorno al correr
ASPNETCORE_ENVIRONMENT=Production dotnet MiApi.dll

# Con dotnet run
dotnet run --environment Production

# Variables de entorno que sobreescriben appsettings.json
# El separador es __ (doble guion bajo)
ConnectionStrings__Default="Host=prod-db;..."
Jwt__Secret="valor-inyectado-desde-secret-manager"

# User Secrets (solo desarrollo — nunca en producción)
dotnet user-secrets init
dotnet user-secrets set "Jwt:Secret" "clave-de-desarrollo"
dotnet user-secrets list
```

