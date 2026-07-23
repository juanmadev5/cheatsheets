# C# — Herramientas

---

## 📌 dotnet CLI

```bash
# Crear proyectos
dotnet new console -n MyApp                # Aplicación de consola
dotnet new classlib -n MyLibrary            # Librería de clases
dotnet new xunit -n MyApp.Tests             # Proyecto de tests con xUnit
dotnet new sln -n MySolution                # Solución vacía

# Solución — agrupar múltiples proyectos
dotnet sln add MyApp/MyApp.csproj
dotnet sln add MyApp.Tests/MyApp.Tests.csproj

# Compilar y ejecutar
dotnet build
dotnet run --project MyApp
dotnet watch run                            # Hot reload en desarrollo

# Publicar
dotnet publish -c Release -o ./publish
dotnet publish -c Release -r linux-x64 --self-contained true   # Self-contained para un runtime específico

# Testing
dotnet test
dotnet test --filter "FullyQualifiedName~UserServiceTests"

# Paquetes NuGet
dotnet add package Newtonsoft.Json
dotnet add package Microsoft.EntityFrameworkCore --version 10.0.0
dotnet remove package Newtonsoft.Json
dotnet list package
dotnet list package --outdated

# Referencias entre proyectos
dotnet add MyApp/MyApp.csproj reference MyLibrary/MyLibrary.csproj

# Herramientas globales y locales
dotnet tool install --global dotnet-ef
dotnet tool exec dotnet-ef migrations add InitialCreate   # Ejecutar sin instalar (.NET 10+)
dnx dotnet-ef database update                              # Script de ejecución directa (.NET 10+)

# Formateo y análisis
dotnet format
dotnet --info                               # SDKs y runtimes instalados
dotnet --list-sdks
```

---

## 📄 El archivo .csproj

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net10.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <RootNamespace>MyApp</RootNamespace>
    <LangVersion>latest</LangVersion>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.EntityFrameworkCore" Version="10.0.0" />
    <PackageReference Include="FluentValidation" Version="12.0.0" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\MyLibrary\MyLibrary.csproj" />
  </ItemGroup>

</Project>
```

* **`TargetFramework`**: versión del runtime objetivo (`net10.0`, `net8.0`, etc.).
* **`ImplicitUsings`**: agrega automáticamente los `using` más comunes del BCL (`System`, `System.Linq`, etc.).
* **`Nullable`**: habilita el análisis de Nullable Reference Types (ver [nullable/nullable.md](../nullable/nullable.md)).
* **`PackageReference`**: dependencias NuGet, gestionadas también por `dotnet add package`.

---

## 📦 Archivos de un proyecto .NET

| Archivo | Propósito |
|---|---|
| **`.csproj`** | Configuración del proyecto — SDK, target framework, dependencias |
| **`.sln`** | Agrupa múltiples proyectos relacionados |
| **`Directory.Build.props`** | Propiedades MSBuild compartidas por todos los proyectos de una carpeta |
| **`global.json`** | Fija la versión del SDK de .NET a usar en el repo |
| **`nuget.config`** | Fuentes de paquetes NuGet personalizadas |
| **`appsettings.json`** | Configuración de la app (ASP.NET Core) |

```json
// global.json — fijar versión exacta del SDK
{
  "sdk": {
    "version": "10.0.100",
    "rollForward": "latestMinor"
  }
}
```

---

## 🧪 Testing con xUnit

```csharp
// MyApp.Tests.csproj requiere OutputType Exe con xUnit v3
using Xunit;

public class CalculatorTests
{
    [Fact]
    public void Add_ReturnsSum_WhenGivenTwoNumbers()
    {
        var calculator = new Calculator();
        int result = calculator.Add(2, 3);
        Assert.Equal(5, result);
    }

    [Theory]
    [InlineData(2, 3, 5)]
    [InlineData(-1, 1, 0)]
    [InlineData(0, 0, 0)]
    public void Add_ReturnsSum_ForVariousInputs(int a, int b, int expected)
    {
        var calculator = new Calculator();
        Assert.Equal(expected, calculator.Add(a, b));
    }

    [Fact]
    public async Task FetchDataAsync_ReturnsExpectedResult()
    {
        var service = new DataService();
        var result = await service.FetchDataAsync();
        Assert.NotNull(result);
    }
}

// IAsyncLifetime — setup/teardown asíncrono (xUnit v3 usa ValueTask)
public class DatabaseTests : IAsyncLifetime
{
    public async ValueTask InitializeAsync() => await SetupDatabaseAsync();
    public async ValueTask DisposeAsync() => await CleanupDatabaseAsync();
}
```

```bash
dotnet add package xunit.v3
dotnet add package xunit.runner.visualstudio
dotnet add package Moq            # Mocking
dotnet add package FluentAssertions --version 7.*   # v8+ requiere licencia comercial
```

---

## 🛠️ Buenas prácticas

* **`Directory.Build.props`** para compartir configuración (nullable, langversion, análisis) entre todos los proyectos de una solución.
* **`global.json`** en repos de equipo — evita discrepancias de versión de SDK entre desarrolladores.
* **`dotnet format`** en CI para mantener estilo consistente sin depender de la config del editor de cada desarrollador.
* **Un proyecto de test por proyecto de producción** (`MyApp` → `MyApp.Tests`), reflejando namespaces y carpetas.
* **`dotnet watch run`** durante desarrollo local — recompila y reinicia automáticamente ante cambios.
