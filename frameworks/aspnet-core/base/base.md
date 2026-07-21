# ASP.NET Core — Base

---

## ⚙️ Proyecto — csproj y estructura

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Project Sdk="Microsoft.NET.Sdk.Web">

    <PropertyGroup>
        <TargetFramework>net10.0</TargetFramework>
        <Nullable>enable</Nullable>
        <ImplicitUsings>enable</ImplicitUsings>
        <InvariantGlobalization>true</InvariantGlobalization>
        <UserSecretsId>a1b2c3d4-e5f6-7890-abcd-ef1234567890</UserSecretsId>
        <DockerDefaultTargetOS>Linux</DockerDefaultTargetOS>
    </PropertyGroup>

    <ItemGroup>
        <!-- ===== ENTITY FRAMEWORK CORE ===== -->
        <PackageReference Include="Microsoft.EntityFrameworkCore" Version="10.0.10" />
        <PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="10.0.10">
            <PrivateAssets>all</PrivateAssets>
        </PackageReference>
        <PackageReference Include="Npgsql.EntityFrameworkCore.PostgreSQL" Version="10.0.3" />

        <!-- ===== VALIDACIÓN ===== -->
        <!-- Solo el paquete core: FluentValidation.AspNetCore (auto-validation en el pipeline MVC)
             ya no se recomienda — no es async y no soporta Minimal APIs. La validación se invoca
             manualmente con IValidator<T> (ver Web API — Validación) -->
        <PackageReference Include="FluentValidation" Version="12.1.1" />

        <!-- ===== AUTENTICACIÓN / SEGURIDAD ===== -->
        <PackageReference Include="Microsoft.AspNetCore.Authentication.JwtBearer" Version="10.0.10" />
        <PackageReference Include="Microsoft.AspNetCore.Identity.EntityFrameworkCore" Version="10.0.10" />

        <!-- ===== MAPEO ===== -->
        <PackageReference Include="Mapster" Version="10.0.11" />

        <!-- ===== CACHÉ ===== -->
        <PackageReference Include="Microsoft.Extensions.Caching.StackExchangeRedis" Version="10.0.10" />

        <!-- ===== MEDIATR (eventos / CQRS) ===== -->
        <!-- Licencia dual RPL-1.5 / comercial desde la v13 (jul 2025) — gratuita para individuos
             y empresas con facturación anual menor a USD 5M (ver mediatr.io) -->
        <PackageReference Include="MediatR" Version="14.2.0" />

        <!-- ===== SCHEDULING ===== -->
        <PackageReference Include="Quartz.Extensions.Hosting" Version="3.18.2" />

        <!-- ===== MAIL ===== -->
        <PackageReference Include="MailKit" Version="4.17.0" />

        <!-- ===== OPENAPI ===== -->
        <!-- Swashbuckle ya no se usa por defecto: Microsoft.AspNetCore.OpenApi (generación) +
             Scalar.AspNetCore (UI interactiva) son la vía recomendada desde .NET 9/10 -->
        <PackageReference Include="Microsoft.AspNetCore.OpenApi" Version="10.0.10" />
        <PackageReference Include="Scalar.AspNetCore" Version="2.16.16" />

        <!-- ===== HEALTH CHECKS ===== -->
        <PackageReference Include="AspNetCore.HealthChecks.NpgSql" Version="9.0.0" />
        <PackageReference Include="AspNetCore.HealthChecks.Redis" Version="9.0.0" />

        <!-- ===== TESTING ===== -->
        <PackageReference Include="Microsoft.AspNetCore.Mvc.Testing" Version="10.0.10" />
        <PackageReference Include="xunit.v3" Version="3.2.2" />
        <PackageReference Include="xunit.runner.visualstudio" Version="3.1.5" />
        <PackageReference Include="Moq" Version="4.20.72" />
        <!-- FluentAssertions 8+ exige licencia comercial de pago (Xceed) — se fija en la
             última versión 7.x, que sigue siendo Apache 2.0 gratuita -->
        <PackageReference Include="FluentAssertions" Version="7.2.2" />
        <PackageReference Include="Testcontainers.PostgreSql" Version="4.13.0" />
    </ItemGroup>

</Project>
```

---

## 📁 Estructura de proyecto

```
src/
├── MiApi/
│   ├── Program.cs                          # Punto de entrada + configuración
│   ├── appsettings.json
│   ├── appsettings.Development.json
│   ├── appsettings.Production.json
│   ├── Endpoints/
│   │   ├── ProductEndpoints.cs
│   │   └── AuthEndpoints.cs
│   ├── Controllers/                        # Si se usa MVC en lugar de Minimal APIs
│   │   ├── ProductController.cs
│   │   └── AuthController.cs
│   ├── Services/
│   │   ├── IProductService.cs
│   │   ├── ProductService.cs
│   │   └── AuthService.cs
│   ├── Repositories/
│   │   ├── IProductRepository.cs
│   │   └── ProductRepository.cs
│   ├── Entities/
│   │   ├── Product.cs
│   │   ├── User.cs
│   │   └── BaseEntity.cs
│   ├── Dtos/
│   │   ├── Requests/
│   │   │   ├── CreateProductRequest.cs
│   │   │   └── LoginRequest.cs
│   │   └── Responses/
│   │       ├── ProductResponse.cs
│   │       ├── PageResponse.cs
│   │       └── ApiResponse.cs
│   ├── Mapping/
│   │   └── ProductMappingConfig.cs
│   ├── Security/
│   │   ├── JwtService.cs
│   │   └── CurrentUserService.cs
│   ├── Data/
│   │   ├── AppDbContext.cs
│   │   ├── Configurations/
│   │   │   └── ProductConfiguration.cs
│   │   └── Migrations/
│   ├── Exceptions/
│   │   ├── ResourceNotFoundException.cs
│   │   └── BusinessException.cs
│   ├── Validators/
│   │   └── CreateProductRequestValidator.cs
│   └── Common/
│       └── PageUtils.cs
└── MiApi.Tests/
    ├── Unit/
    │   ├── ProductServiceTests.cs
    │   └── ProductRepositoryTests.cs
    └── Integration/
        └── ProductEndpointsTests.cs
```

---

## 🚀 Program.cs — configuración completa

```csharp
using System.Text;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.OpenApi;
using Microsoft.EntityFrameworkCore;
using Microsoft.IdentityModel.Tokens;
using Microsoft.OpenApi;
using FluentValidation;
using MediatR;
using Quartz;
using Scalar.AspNetCore;
using Serilog;

var builder = WebApplication.CreateBuilder(args);

// ===== SERILOG =====
builder.Host.UseSerilog((context, config) =>
    config.ReadFrom.Configuration(context.Configuration));

// ===== CONFIGURATION OPTIONS =====
builder.Services.Configure<JwtOptions>(
    builder.Configuration.GetSection("Jwt"));
builder.Services.Configure<CorsOptions>(
    builder.Configuration.GetSection("Cors"));

// ===== DBCONTEXT =====
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("Default"),
        npgsql => npgsql.EnableRetryOnFailure(3)));

// ===== IDENTITY =====
builder.Services.AddIdentityCore<ApplicationUser>(options =>
    {
        options.Password.RequiredLength = 8;
        options.User.RequireUniqueEmail = true;
    })
    .AddRoles<IdentityRole<Guid>>()
    .AddEntityFrameworkStores<AppDbContext>()
    .AddDefaultTokenProviders();

// ===== AUTENTICACIÓN JWT =====
var jwtSection = builder.Configuration.GetSection("Jwt");
builder.Services.AddAuthentication(options =>
    {
        options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
        options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
    })
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = jwtSection["Issuer"],
            ValidAudience = jwtSection["Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(jwtSection["Secret"]!)),
            ClockSkew = TimeSpan.Zero
        };
    });

// ===== AUTORIZACIÓN =====
builder.Services.AddAuthorizationBuilder()
    .AddPolicy("AdminOnly", policy => policy.RequireRole("Admin"))
    .AddPolicy("MinimumAge", policy => policy.RequireClaim("age"));

// ===== CORS =====
builder.Services.AddCors(options =>
{
    options.AddPolicy("Default", policy =>
    {
        policy.WithOrigins(
                builder.Configuration.GetSection("Cors:AllowedOrigins").Get<string[]>() ?? [])
            .AllowAnyMethod()
            .AllowAnyHeader()
            .AllowCredentials();
    });
});

// ===== CONTROLLERS / MINIMAL APIS =====
builder.Services.AddControllers();

// ===== OPENAPI =====
builder.Services.AddOpenApi(options =>
{
    options.AddDocumentTransformer((document, context, cancellationToken) =>
    {
        document.Info = new() { Title = "Mi API", Version = "v1" };
        return Task.CompletedTask;
    });
    options.AddDocumentTransformer<BearerSecuritySchemeTransformer>();
});

// ===== VALIDACIÓN =====
// La validación se invoca manualmente en el endpoint/Service con IValidator<T> (ver Web
// API — Validación). FluentValidation.AspNetCore (auto-validation) ya no se recomienda.
builder.Services.AddValidatorsFromAssemblyContaining<Program>();

// ===== MAPSTER =====
builder.Services.AddMapster();

// ===== MEDIATR =====
builder.Services.AddMediatR(cfg =>
    cfg.RegisterServicesFromAssemblyContaining<Program>());

// ===== CACHÉ =====
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration.GetConnectionString("Redis");
    options.InstanceName = "MiApi:";
});

// ===== QUARTZ =====
builder.Services.AddQuartz(q =>
{
    var jobKey = new JobKey("CleanupJob");
    q.AddJob<CleanupJob>(opts => opts.WithIdentity(jobKey));
    q.AddTrigger(opts => opts
        .ForJob(jobKey)
        .WithCronSchedule("0 0 2 * * ?"));
});
builder.Services.AddQuartzHostedService(options => options.WaitForJobsToComplete = true);

// ===== HEALTH CHECKS =====
builder.Services.AddHealthChecks()
    .AddNpgSql(builder.Configuration.GetConnectionString("Default")!)
    .AddRedis(builder.Configuration.GetConnectionString("Redis")!);

// ===== SERVICIOS DE LA APLICACIÓN =====
builder.Services.AddScoped<IProductService, ProductService>();
builder.Services.AddScoped<IProductRepository, ProductRepository>();
builder.Services.AddScoped<IAuthService, AuthService>();
builder.Services.AddScoped<ICurrentUserService, CurrentUserService>();
builder.Services.AddHttpContextAccessor();

// ===== RATE LIMITING =====
builder.Services.AddRateLimiter(options =>
{
    options.AddFixedWindowLimiter("Default", opt =>
    {
        opt.PermitLimit = 100;
        opt.Window = TimeSpan.FromMinutes(1);
    });
});

// ===== PROBLEM DETAILS =====
builder.Services.AddProblemDetails();
builder.Services.AddExceptionHandler<GlobalExceptionHandler>();

var app = builder.Build();

// ===== MIDDLEWARE PIPELINE =====
if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
    app.MapScalarApiReference();   // UI interactiva en /scalar
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler();
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseSerilogRequestLogging();
app.UseCors("Default");
app.UseRateLimiter();
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
app.MapProductEndpoints();     // Extension method con Minimal APIs
app.MapAuthEndpoints();
app.MapHealthChecks("/health");

app.Run();

// Agrega el esquema de seguridad Bearer al documento OpenAPI generado
internal sealed class BearerSecuritySchemeTransformer(IAuthenticationSchemeProvider authenticationSchemeProvider)
    : IOpenApiDocumentTransformer
{
    public async Task TransformAsync(OpenApiDocument document, OpenApiDocumentTransformerContext context, CancellationToken cancellationToken)
    {
        var schemes = await authenticationSchemeProvider.GetAllSchemesAsync();
        if (schemes.Any(s => s.Name == JwtBearerDefaults.AuthenticationScheme))
        {
            document.Components ??= new OpenApiComponents();
            document.Components.SecuritySchemes = new Dictionary<string, IOpenApiSecurityScheme>
            {
                ["Bearer"] = new OpenApiSecurityScheme
                {
                    Type = SecuritySchemeType.Http,
                    Scheme = "bearer",
                    BearerFormat = "JWT"
                }
            };
        }
    }
}

// Necesario para WebApplicationFactory en tests de integración
public partial class Program { }
```

---

## ⚙️ appsettings.json — configuración completa

```json
{
  "ConnectionStrings": {
    "Default": "Host=localhost;Port=5432;Database=mi_db;Username=postgres;Password=secret",
    "Redis": "localhost:6379"
  },
  "Jwt": {
    "Secret": "mi-clave-super-secreta-de-256-bits-para-hs256",
    "Issuer": "mi-api",
    "Audience": "mi-app",
    "ExpirationMinutes": 60,
    "RefreshExpirationDays": 7
  },
  "Cors": {
    "AllowedOrigins": [ "http://localhost:3000", "https://miapp.com" ]
  },
  "Upload": {
    "Directory": "/var/app/uploads",
    "MaxSizeBytes": 10485760,
    "AllowedTypes": [ "image/jpeg", "image/png", "application/pdf" ]
  },
  "Serilog": {
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft.AspNetCore": "Warning",
        "Microsoft.EntityFrameworkCore": "Warning"
      }
    },
    "WriteTo": [
      { "Name": "Console" },
      {
        "Name": "File",
        "Args": { "path": "logs/log-.txt", "rollingInterval": "Day" }
      }
    ]
  },
  "AllowedHosts": "*"
}
```

```json
// appsettings.Development.json
{
  "Serilog": {
    "MinimumLevel": { "Default": "Debug" }
  },
  "Logging": {
    "LogLevel": {
      "Microsoft.EntityFrameworkCore.Database.Command": "Information"
    }
  }
}
```

```json
// appsettings.Production.json
{
  "Serilog": {
    "MinimumLevel": { "Default": "Warning" }
  }
}
```

```bash
# Variables de entorno equivalentes (sobreescriben appsettings.json)
ConnectionStrings__Default="Host=prod-db;..."
Jwt__Secret="valor-inyectado-desde-secret-manager"
ASPNETCORE_ENVIRONMENT=Production
```

---


## ⚙️ Comandos dotnet CLI esenciales

```bash
# Crear un nuevo proyecto Web API
dotnet new webapi -n MiApi

# Restaurar dependencias
dotnet restore

# Compilar
dotnet build

# Ejecutar tests
dotnet test

# Correr la aplicación
dotnet run

# Correr con hot reload (desarrollo)
dotnet watch run

# Correr con perfil de entorno específico
dotnet run --environment Production

# Publicar para producción
dotnet publish -c Release -o ./publish

# Agregar un paquete NuGet
dotnet add package Nombre.Del.Paquete

# Ver árbol de dependencias
dotnet list package

# Ver paquetes desactualizados
dotnet list package --outdated

# Migraciones EF Core
dotnet ef migrations add NombreMigracion
dotnet ef database update

# Formatear código según .editorconfig
dotnet format
```

