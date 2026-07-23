# C# — Programación Orientada a Objetos

---

## 📌 Clases

```csharp
public class Vehicle
{
    // Campos privados
    private int _speed;

    // Propiedades — auto-implementadas
    public string Brand { get; set; }
    public string Model { get; set; }
    public int Speed
    {
        get => _speed;
        set => _speed = value < 0 ? 0 : value;   // Validación en el setter
    }

    // Propiedad de solo lectura calculada
    public string FullName => $"{Brand} {Model}";

    // Constructor
    public Vehicle(string brand, string model)
    {
        Brand = brand;
        Model = model;
    }

    // Constructor sin parámetros que delega en el principal
    public Vehicle() : this("Desconocido", "Desconocido") { }

    // Métodos
    public void Accelerate(int amount) => Speed += amount;

    // Override de ToString
    public override string ToString() => $"{FullName} a {Speed} km/h";
}

var car = new Vehicle("Toyota", "Corolla");
car.Accelerate(50);
Console.WriteLine(car);
```

---

## 🧬 Herencia e interfaces

```csharp
// Clase base
public abstract class Shape
{
    public abstract double Area();               // Debe implementarse en la subclase
    public virtual string Describe() => $"Área: {Area():F2}";   // Puede sobreescribirse
}

// Herencia con override
public class Circle : Shape
{
    public double Radius { get; init; }
    public override double Area() => Math.PI * Radius * Radius;
    public override string Describe() => $"Círculo — {base.Describe()}";   // base llama a la implementación del padre
}

// Interfaces — pueden tener miembros con implementación por defecto (C# 8+)
public interface ILogger
{
    void Log(string message);
    void LogError(string message) => Log($"[ERROR] {message}");   // Default interface method
}

public class ConsoleLogger : ILogger
{
    public void Log(string message) => Console.WriteLine(message);
    // LogError se hereda de la implementación default
}

// Implementación de múltiples interfaces
public interface IDisposableResource : IDisposable, IAsyncDisposable { }

// sealed — evita herencia adicional
public sealed class FinalImplementation : Shape
{
    public override double Area() => 0;
}

// Chequeo y cast de tipos
Shape shape = new Circle { Radius = 5 };
if (shape is Circle circle) Console.WriteLine(circle.Radius);
```

---

## 🏛️ Modificadores de clase

| Modificador | Significado |
|---|---|
| **`abstract`** | No se puede instanciar; puede tener miembros sin implementación. |
| **`sealed`** | No se puede heredar. |
| **`static`** | Solo miembros estáticos, no se puede instanciar. |
| **`partial`** | La clase se divide en varios archivos (útil con generadores de código). |
| **`internal`** | Visible solo dentro del mismo assembly (default si no se especifica). |
| **`public` / `private` / `protected` / `protected internal` / `private protected`** | Niveles de acceso estándar. |

```csharp
// static class — solo miembros estáticos
public static class MathUtils
{
    public static int Square(int x) => x * x;
}

// partial class — se puede combinar con generadores de código (p. ej. source generators)
public partial class Product
{
    public string Name { get; set; }
}
public partial class Product
{
    public decimal Price { get; set; }
}
```

---

## 📐 Structs

```csharp
// struct — value type, se copia al asignar o pasar por valor
public struct Point
{
    public double X { get; init; }
    public double Y { get; init; }

    public double DistanceTo(Point other) =>
        Math.Sqrt(Math.Pow(X - other.X, 2) + Math.Pow(Y - other.Y, 2));
}

var p1 = new Point { X = 0, Y = 0 };
var p2 = p1;             // Copia por valor — p2 es independiente de p1

// readonly struct — inmutable, mejora rendimiento al evitar copias defensivas
public readonly struct Money(decimal amount, string currency)
{
    public decimal Amount { get; } = amount;
    public string Currency { get; } = currency;
}

// record struct — igualdad estructural + sintaxis concisa (C# 10+)
public record struct Coordinate(double Lat, double Lng);
```

---

## 🎛️ Propiedades e indexers

```csharp
public class Matrix
{
    private readonly double[,] _data = new double[3, 3];

    // Indexer — acceso tipo array a una instancia
    public double this[int row, int col]
    {
        get => _data[row, col];
        set => _data[row, col] = value;
    }
}

var m = new Matrix();
m[0, 0] = 5.0;
Console.WriteLine(m[0, 0]);

// Propiedades estáticas y de solo inicialización
public class AppConfig
{
    public static AppConfig Default { get; } = new();
    public string Environment { get; init; } = "Production";
}
```

---

## ⚙️ Companion estático, singleton y factory

```csharp
// Miembros estáticos — equivalente a companion object de Kotlin
public class UserFactory
{
    private static int _nextId = 1;

    public static User Create(string name) => new(_nextId++, name);
}

// Singleton thread-safe con Lazy<T>
public sealed class Cache
{
    private static readonly Lazy<Cache> _instance = new(() => new Cache());
    public static Cache Instance => _instance.Value;

    private Cache() { }
}
```

---

## ➕ Operator overloading

```csharp
public readonly struct Vector2(double x, double y)
{
    public double X { get; } = x;
    public double Y { get; } = y;

    public static Vector2 operator +(Vector2 a, Vector2 b) => new(a.X + b.X, a.Y + b.Y);
    public static Vector2 operator -(Vector2 a, Vector2 b) => new(a.X - b.X, a.Y - b.Y);
    public static bool operator ==(Vector2 a, Vector2 b) => a.X == b.X && a.Y == b.Y;
    public static bool operator !=(Vector2 a, Vector2 b) => !(a == b);

    public override bool Equals(object? obj) => obj is Vector2 other && this == other;
    public override int GetHashCode() => HashCode.Combine(X, Y);
}

var sum = new Vector2(1, 2) + new Vector2(3, 4);
```

---

## 🛠️ Buenas prácticas

* **`init` en vez de `set`** para propiedades que solo deben asignarse en la construcción.
* **`sealed`** por defecto en clases que no están diseñadas explícitamente para herencia.
* **Interfaces pequeñas y específicas** (Interface Segregation) en vez de una única interfaz con muchos métodos.
* **`readonly struct`** para value types inmutables — evita copias defensivas del compilador.
* **Composición sobre herencia** cuando la relación no es estrictamente "es un".
