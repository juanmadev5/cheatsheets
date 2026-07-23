# C# — C# Moderno (9 a 14 / .NET 5 a 10)

---

## 📌 Novedades del lenguaje por versión

Resumen de las features más usadas introducidas desde C# 9. La versión de lenguaje la determina el SDK de .NET usado (C# 14 requiere .NET 10).

| Versión | .NET | Features clave |
|---|---|---|
| **C# 9** | .NET 5 | `record`, top-level statements, init-only setters, pattern matching con relacionales |
| **C# 10** | .NET 6 | file-scoped namespaces, `record struct`, global usings |
| **C# 11** | .NET 7 | raw string literals, required members, generic math, list patterns |
| **C# 12** | .NET 8 | primary constructors (cualquier clase/struct), collection expressions, alias any type |
| **C# 13** | .NET 9 | `params` collections, nuevo tipo `Lock`, partial properties/indexers |
| **C# 14** | .NET 10 | `field` keyword, extension members, null-conditional assignment, implicit span conversions |

---

## 🧾 Top-level statements y file-scoped namespaces

```csharp
// Program.cs completo — sin Main() explícito ni llaves de clase (C# 9+)
using System;

Console.WriteLine("Hola mundo");

var numbers = new[] { 1, 2, 3 };
foreach (var n in numbers) Console.WriteLine(n);

return 0;
```

```csharp
// File-scoped namespace (C# 10+) — reemplaza el bloque { } con indentación
namespace MyApp.Services;

public class UserService
{
    // ...
}
```

```csharp
// Global usings (C# 10+) — en un archivo GlobalUsings.cs, aplican a todo el proyecto
global using System;
global using System.Collections.Generic;
global using System.Linq;
```

---

## 📦 Records

`record` genera igualdad estructural, `ToString()` legible y soporte para `with` — ideal para modelar datos inmutables.

```csharp
// record class — reference type con igualdad por valor
public record Person(string Name, int Age);

var p1 = new Person("Ana", 30);
var p2 = new Person("Ana", 30);
Console.WriteLine(p1 == p2);        // true — igualdad estructural
Console.WriteLine(p1);              // Person { Name = Ana, Age = 30 }

// with — crea una copia con algunos valores modificados (non-destructive mutation)
var p3 = p1 with { Age = 31 };

// record struct (C# 10+) — value type con la misma ergonomía
public record struct Point(double X, double Y);

// record con cuerpo extendido — métodos y validación adicional
public record Order(string Id, decimal Total)
{
    public bool IsValid => Total > 0;

    public Order(string id, decimal total, string currency) : this(id, total)
    {
        Currency = currency;
    }

    public string Currency { get; init; } = "USD";
}

// Deconstrucción automática
var (name, age) = p1;

// Records inmutables — las propiedades generadas son { get; init; }
// p1.Age = 40;   // Error de compilación
```

---

## 🏗️ Primary constructors (C# 12+)

```csharp
// Antes — constructor explícito con campos manuales
public class UserServiceOld
{
    private readonly IRepository _repository;
    private readonly ILogger _logger;

    public UserServiceOld(IRepository repository, ILogger logger)
    {
        _repository = repository;
        _logger = logger;
    }
}

// C# 12+ — primary constructor en cualquier class o struct (no solo records)
public class UserService(IRepository repository, ILogger logger)
{
    public void Register(string name)
    {
        logger.Log($"Registrando {name}");   // Los parámetros están en scope de toda la clase
        repository.Save(name);
    }
}

// Con propiedades explícitas a partir de los parámetros (no son públicas por defecto, a diferencia de record)
public class Point(double x, double y)
{
    public double X { get; } = x;
    public double Y { get; } = y;
    public double Magnitude => Math.Sqrt(X * X + Y * Y);
}
```

---

## 🧩 Pattern matching avanzado

```csharp
// Type patterns con declaración
object obj = "hola";
if (obj is string text) Console.WriteLine(text.ToUpper());

// Property patterns
if (order is { Total: > 100, Status: "Pending" })
{
    Console.WriteLine("Pedido grande pendiente");
}

// Nested property patterns
if (user is { Address.City: "Buenos Aires" })
{
    Console.WriteLine("Usuario porteño");
}

// Positional patterns (requieren Deconstruct, records lo generan automático)
if (point is (0, 0)) Console.WriteLine("Origen");

// List patterns (C# 11+)
int[] numbers = [1, 2, 3, 4, 5];
if (numbers is [1, 2, ..])       Console.WriteLine("Empieza con 1, 2");
if (numbers is [.., 4, 5])       Console.WriteLine("Termina con 4, 5");
if (numbers is [var first, .., var last]) Console.WriteLine($"{first}..{last}");

// Relational y lógicos combinados (C# 9+)
string Classify(int age) => age switch
{
    < 0             => "Inválido",
    >= 0 and < 18   => "Menor",
    >= 18 and < 65  => "Adulto",
    _               => "Mayor"
};

// Combinación con and / or / not
if (obj is not null and (int or long)) Console.WriteLine("Es un entero");
```

---

## 📥 Collection expressions (C# 12+)

```csharp
// Sintaxis unificada [ ] para arrays, List<T>, Span<T>, etc.
int[] array = [1, 2, 3, 4, 5];
List<string> names = ["Ana", "Beto", "Carla"];
Span<char> chars = ['a', 'b', 'c'];

// Spread element — inline de otra colección con ..
int[] row0 = [1, 2, 3];
int[] row1 = [4, 5, 6];
int[] combined = [.. row0, .. row1, 7, 8, 9];

// Arrays multidimensionales (jagged)
int[][] matrix = [[1, 2], [3, 4], [5, 6]];

// params con Span<T> (C# 13+) — más eficiente que params T[]
void PrintAll(params ReadOnlySpan<string> items)
{
    foreach (var item in items) Console.WriteLine(item);
}
PrintAll("a", "b", "c");
```

---

## 🔑 Required members y init-only setters

```csharp
public class UserDto
{
    public required string Name { get; init; }   // Obligatorio al inicializar (C# 11+)
    public required string Email { get; init; }
    public int Age { get; init; }                 // init — solo asignable en el inicializador

    // Constructor con SetsRequiredMembers si se provee uno propio
    [System.Diagnostics.CodeAnalysis.SetsRequiredMembers]
    public UserDto(string name, string email) => (Name, Email) = (name, email);
}

var user = new UserDto { Name = "Ana", Email = "ana@example.com" };
// var invalid = new UserDto { Name = "Ana" };   // Error: falta Email (required)
```

---

## 🧵 El `field` keyword (C# 14)

```csharp
// Antes — backing field manual para agregar lógica al setter
private string _message = "";
public string Message
{
    get => _message;
    set => _message = value ?? throw new ArgumentNullException(nameof(value));
}

// C# 14 — field accede al backing field generado por el compilador
public string Message
{
    get;
    set => field = value ?? throw new ArgumentNullException(nameof(value));
}

// Propiedad con valor por defecto y validación, sin declarar campo propio
public int Age
{
    get;
    set => field = value >= 0 ? value : throw new ArgumentOutOfRangeException(nameof(value));
} = 0;
```

---

## 🧬 Extension members (C# 14)

```csharp
// Sintaxis moderna con bloque extension — permite propiedades y miembros estáticos
public static class EnumerableExtensions
{
    extension<T>(IEnumerable<T> source)
    {
        // Extension property
        public bool IsEmpty => !source.Any();

        // Extension method (equivalente a la sintaxis clásica con "this")
        public IEnumerable<T> WhereNotNull() => source.Where(x => x is not null);
    }

    extension<T>(IEnumerable<T>)   // Solo el tipo receptor — miembros estáticos
    {
        public static IEnumerable<T> Empty() => [];
    }
}

// Uso
var list = new List<int?> { 1, null, 2 };
bool empty = list.IsEmpty;

// Sintaxis clásica de extension methods (sigue vigente y es la más común)
public static class StringExtensions
{
    public static string Truncate(this string value, int maxLength) =>
        value.Length <= maxLength ? value : value[..maxLength] + "...";
}
"Hola mundo".Truncate(4);   // "Hola..."
```

---

## 🛠️ Buenas prácticas

* **`record`** para DTOs y modelos inmutables — evita boilerplate de `Equals`/`GetHashCode`/`ToString`.
* **Primary constructors** para inyección de dependencias simple — reduce ruido en servicios y controllers.
* **Collection expressions `[...]`** en vez de `new List<T> { ... }` — sintaxis más uniforme entre tipos de colección.
* **`required`** en vez de constructores largos para DTOs con muchas propiedades obligatorias.
* **List patterns y property patterns** en `switch` para reemplazar cadenas de `if` anidados.
