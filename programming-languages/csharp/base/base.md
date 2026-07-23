# C# — Base

---

## 📦 Namespaces y using

Los namespaces organizan tipos en contenedores lógicos (no necesariamente ligados a la estructura de carpetas, a diferencia de Java/Kotlin) y evitan colisiones de nombres entre librerías.

```csharp
// Declaración clásica en bloque — puede haber varios en el mismo archivo
namespace MyApp.Services
{
    public class UserService { }
}

namespace MyApp.Models
{
    public class User { }
}

// File-scoped namespace (C# 10+) — un solo namespace por archivo, sin indentación extra
// Ver csharp-moderno.md para más detalle
namespace MyApp.Services;

public class UserService { }

// Namespaces anidados — acceso con el nombre completo
namespace Company.Product.Feature
{
    public class Widget { }
}
var widget = new Company.Product.Feature.Widget();
```

```csharp
// using — importa los tipos de un namespace para usarlos sin calificar
using System;
using System.Collections.Generic;
using MyApp.Services;

var service = new UserService();   // Sin necesidad de MyApp.Services.UserService

// using static — importa los miembros estáticos de un tipo directamente
using static System.Math;
using static System.Console;

WriteLine(Sqrt(16));   // En vez de Console.WriteLine(Math.Sqrt(16))

// using alias — renombra un tipo o namespace, útil para desambiguar colisiones
using Project = MyApp.Models.Project;
using LegacyUser = MyApp.Legacy.User;

// Alias de cualquier tipo, incluyendo genéricos y tuplas (C# 12+)
using StringPair = (string First, string Second);

// Uso completamente calificado sin using — siempre válido como alternativa
var user = new MyApp.Models.User();
```

* **`global using`** (C# 10+) y **global usings implícitos** (`<ImplicitUsings>enable</ImplicitUsings>`) evitan repetir los `using` más comunes en cada archivo — ver [csharp-moderno.md](../csharp-moderno/csharp-moderno.md) y [herramientas.md](../herramientas/herramientas.md).
* El namespace **no tiene por qué coincidir con la carpeta física** del archivo, aunque Visual Studio/`dotnet new` lo hacen por convención (`RootNamespace` + ruta relativa).
* **`internal`** (modificador de acceso, ver [poo.md](../poo/poo.md)) limita la visibilidad al *assembly*, no al namespace — dos clases `internal` en namespaces distintos del mismo proyecto sí pueden verse entre sí.

---

## 📌 Variables y constantes

```csharp
// var — inferencia de tipo (el tipo se fija en tiempo de compilación, no es dynamic)
var name = "Juan";
var age = 28;
var pi = 3.14159;

// Declaración explícita de tipo
string city = "Buenos Aires";
int count = 0;
double price = 19.99;

// const — constante en tiempo de compilación (debe inicializarse al declarar)
const int MaxRetries = 3;
const string BaseUrl = "https://api.example.com";
const double Pi = 3.14159;

// readonly — se asigna una sola vez, en la declaración o en el constructor
class Config
{
    public readonly string Environment;
    public readonly DateTime CreatedAt = DateTime.UtcNow;

    public Config(string environment)
    {
        Environment = environment;   // Válido solo dentro del constructor
    }
}

// static readonly — equivalente a const pero evaluado en runtime (útil para tipos no primitivos)
static readonly HttpClient Client = new();

// Separador de dígitos para legibilidad
const long TimeoutMs = 5_000L;
const int Big = 1_000_000;

// Múltiple asignación con tuplas
var (x, y) = (10, 20);
(int id, string name2, string email) = GetUser();

// Deconstrucción de tipos con Deconstruct()
var person = new Person("Ana", 30);
var (personName, personAge) = person;   // Requiere método Deconstruct en Person
```

---

## 📊 Tipos de datos

### Numéricos

```csharp
// Enteros
sbyte sb = 127;
byte b = 255;
short s = 32_767;
ushort us = 65_535;
int i = 2_147_483_647;
uint ui = 4_294_967_295;
long l = 9_223_372_036_854_775_807L;
ulong ul = 18_446_744_073_709_551_615UL;

// Punto flotante
float f = 3.14f;
double d = 3.141592653589793;
decimal money = 19.99m;    // Precisión exacta — usar siempre para dinero

// Literales especiales
int hex = 0xFF;             // 255
int binary = 0b1010_1010;   // 170

// Conversiones explícitas — C# no convierte automáticamente entre tipos que pierden precisión
int n = 42;
long asLong = n;            // Implícita — int a long no pierde precisión
double asDouble = n;        // Implícita
int back = (int)3.99;       // Explícita (cast) — trunca a 3
string asString = n.ToString();
int parsed = int.Parse("42");
bool ok = int.TryParse("abc", out int result);  // false, result = 0

// checked / unchecked — controlar overflow aritmético
checked
{
    int max = int.MaxValue;
    // max + 1;   // Lanzaría OverflowException dentro de checked
}

// Operaciones numéricas
Console.WriteLine(42 / 5);      // 8 (división entera)
Console.WriteLine(42 / 5.0);    // 8.4
Console.WriteLine(42 % 5);      // 2 (módulo)
Console.WriteLine(Math.Round(3.7));  // 4
Console.WriteLine((int)3.7);         // 3 (trunca, no redondea)

// Infinity y NaN
double.PositiveInfinity
double.NegativeInfinity
double.NaN
double.IsNaN(0.0 / 0.0)
double.IsInfinity(1.0 / 0.0)
```

---

### string, char, bool, object, dynamic

```csharp
// string — inmutable, referencia
string greeting = "Hola mundo";
string multiline = """
    Línea 1
    Línea 2
    Línea 3
    """;                        // Raw string literal (C# 11+)

string interpolated = $"Hola {name}, tenés {age + 1} años";
string verbatim = @"C:\Users\juan\docs";   // Sin necesidad de escapar \

// char
char letter = 'A';
char unicode = '\u0041';    // 'A'

// bool
bool isActive = true;
bool and = true && false;
bool or = true || false;
bool not = !isActive;

// object — supertipo de todos los tipos (value y reference, vía boxing)
object anything = 42;          // Boxing — el int se envuelve en un objeto
int unboxed = (int)anything;   // Unboxing — cast explícito requerido

// dynamic — se resuelve en runtime, sin chequeo del compilador (usar con moderación)
dynamic value = 10;
value = "ahora soy un string";   // Válido — dynamic no tiene tipo fijo

// void — sin valor de retorno
void Greet() => Console.WriteLine("Hola");

// Tipos por valor vs por referencia
struct Point { public int X, Y; }   // Value type — se copia al asignar
class Container { public int Value; } // Reference type — se comparte la referencia
```

---

## 🛡️ Null Safety (Nullable Reference Types)

> Ver detalle completo, incluyendo `?.`, `??`, `!` y el null-conditional assignment de C# 14, en [nullable/nullable.md](../nullable/nullable.md).

```csharp
// Con <Nullable>enable</Nullable> en el .csproj (default en proyectos nuevos desde .NET 6)
string name = "Juan";       // No-nullable — el compilador advierte si podría ser null
string? nickname = null;    // Nullable — explícito con ?

// Nullable<T> para value types
int? maybeAge = null;
DateTime? maybeDate = null;
```

---

## ➕ Operadores

```csharp
// Aritméticos
a + b; a - b; a * b; a / b; a % b;

// Asignación compuesta
a += b; a -= b; a *= b; a /= b; a %= b;

// Incremento / decremento
a++; a--; ++a; --a;

// Comparación
a == b      // Igualdad de valor (para reference types, igualdad referencial salvo override)
a != b
a < b; a > b; a <= b; a >= b;

// Lógicos
a && b; a || b; !a;

// Bit a bit
a & b; a | b; a ^ b; ~a; a << 1; a >> 1;

// Operador ternario
string result = age >= 18 ? "Mayor" : "Menor";

// is / as
if (obj is string text) Console.WriteLine(text.Length);  // Pattern matching con declaración
var maybeText = obj as string;                            // null si el cast falla

// typeof / GetType
Type t1 = typeof(string);
Type t2 = obj.GetType();

// Rangos e índices (C# 8+)
int[] numbers = [1, 2, 3, 4, 5];
Console.WriteLine(numbers[^1]);        // Último elemento — 5
Console.WriteLine(numbers[1..3]);      // Slice — [2, 3]
Console.WriteLine(numbers[..2]);       // Desde el inicio — [1, 2]
Console.WriteLine(numbers[2..]);       // Hasta el final — [3, 4, 5]

// nameof — nombre de un símbolo como string, chequeado por el compilador
throw new ArgumentNullException(nameof(name));

// sizeof (solo tipos primitivos, en contexto unsafe para algunos)
Console.WriteLine(sizeof(int));    // 4
```

---

## 🔀 Control de flujo

### if / switch como expresión

```csharp
// if tradicional
if (score >= 90) Console.WriteLine("Excelente");
else if (score >= 70) Console.WriteLine("Bueno");
else Console.WriteLine("Mejorable");

// Operador ternario como expresión
string label = score >= 90 ? "Excelente" : score >= 70 ? "Bueno" : "Mejorable";

// switch statement clásico
switch (status)
{
    case "active":
        Console.WriteLine("Activo");
        break;
    case "inactive":
    case "disabled":
        Console.WriteLine("Inactivo");
        break;
    default:
        Console.WriteLine($"Desconocido: {status}");
        break;
}

// switch expression (C# 8+) — retorna un valor, exhaustivo
string description = status switch
{
    "active"   => "Activo",
    "inactive" => "Inactivo",
    "pending"  => "Pendiente",
    _          => $"Estado desconocido: {status}"
};

// switch expression con pattern matching de tipos
string Describe(object obj) => obj switch
{
    int n when n < 0  => "Entero negativo",
    int n             => $"Entero: {n}",
    string s          => $"String de longitud {s.Length}",
    null              => "Es null",
    _                 => $"Tipo: {obj.GetType().Name}"
};

// switch con patrones de propiedad y tuplas
string Classify(Point p) => p switch
{
    (0, 0)          => "Origen",
    (var x, 0)      => $"Eje X en {x}",
    (0, var y)      => $"Eje Y en {y}",
    { X: > 0, Y: > 0 } => "Primer cuadrante",
    _               => "Otro cuadrante"
};
```

---

### Bucles

```csharp
// for
for (int i = 0; i < 5; i++) Console.Write(i);

// foreach — sobre cualquier IEnumerable<T>
foreach (var item in list) Console.WriteLine(item);
foreach (var (key, value) in dictionary) Console.WriteLine($"{key} = {value}");

// while
int i = 0;
while (i < 10) { Console.Write(i); i++; }

// do-while
do { ProcessNext(); } while (HasMore());

// break y continue
for (int i = 0; i < 10; i++)
{
    if (i == 3) continue;   // Salta a la siguiente iteración
    if (i == 7) break;      // Sale del bucle
    Console.Write(i);
}

// Bucles anidados con goto (evitar salvo casos muy puntuales — C# no tiene labeled break/continue)
for (int i = 0; i < 5; i++)
{
    for (int j = 0; j < 5; j++)
    {
        if (i == 2 && j == 2) goto End;
    }
}
End:
Console.WriteLine("Salida de bucles anidados");
```

---

## 🛠️ Funciones y métodos

* **`static ReturnType Method(params)`**: Método estático, no requiere instancia.
* **Parámetros opcionales**: `void Log(string msg, string level = "INFO")` — valor por defecto.
* **`params`**: `void Sum(params int[] numbers)` — cantidad variable de argumentos (también soporta `Span<T>` desde C# 13).
* **`ref` / `out` / `in`**: pasar por referencia (`ref` requiere inicialización previa, `out` no, `in` es de solo lectura).
* **Funciones locales**: función anidada dentro de otro método, con acceso a sus variables.
* **Expression-bodied members**: `int Square(int x) => x * x;` — sintaxis concisa para cuerpos de una sola expresión.
