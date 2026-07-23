# C# — Programación Funcional

---

## 📌 Delegates

Un `delegate` es un tipo que representa referencias a métodos con una firma específica — la base de eventos, callbacks y lambdas en C#.

```csharp
// Declaración de un delegate propio
public delegate int Operation(int a, int b);

int Add(int a, int b) => a + b;
int Multiply(int a, int b) => a * b;

Operation op = Add;
Console.WriteLine(op(2, 3));      // 5

op = Multiply;
Console.WriteLine(op(2, 3));      // 6

// Multicast delegates — invoca todos los métodos suscriptos en orden
Action<string> logger = Console.WriteLine;
logger += msg => File.AppendAllText("log.txt", msg);
logger("Evento registrado");   // Llama a ambos
```

---

## 🎯 Func, Action, Predicate

```csharp
// Func<T1, ..., TResult> — hasta 16 parámetros, siempre retorna un valor
Func<int, int, int> add = (a, b) => a + b;
Func<int, int> square = x => x * x;
Func<string> greet = () => "Hola";

// Action<T1, ...> — no retorna valor (equivalente a Func con Unit)
Action<string> print = Console.WriteLine;
Action greetAll = () => Console.WriteLine("Hola a todos");

// Predicate<T> — Func<T, bool> con nombre semántico, usado en List<T>.Find/FindAll
Predicate<int> isEven = n => n % 2 == 0;
List<int> numbers = [1, 2, 3, 4, 5, 6];
List<int> evens = numbers.FindAll(isEven);
```

---

## 🪶 Expresiones lambda

```csharp
// Sintaxis con expresión
Func<int, int, int> sum = (a, b) => a + b;

// Sintaxis con bloque
Func<int, int> factorial = null!;
factorial = n =>
{
    if (n <= 1) return 1;
    return n * factorial(n - 1);
};

// Parámetro único — paréntesis opcionales
Func<int, int> doubleIt = x => x * 2;

// Sin parámetros
Action sayHi = () => Console.WriteLine("Hi");

// Con tipos explícitos
Func<int, int, int> typed = (int a, int b) => a + b;

// Discard de parámetros no usados (C# 9+)
Func<int, int, int> ignoreSecond = (a, _) => a;

// Valores por defecto en lambdas (C# 12+)
var greetOrDefault = (string name = "Anónimo") => $"Hola {name}";

// Modificadores en parámetros sin tipo explícito (C# 14+)
delegate bool TryParse<T>(string text, out T result);
TryParse<int> parse = (text, out result) => int.TryParse(text, out result);

// Closures — capturan variables del contexto
int factor = 3;
Func<int, int> multiplyByFactor = x => x * factor;
factor = 10;
Console.WriteLine(multiplyByFactor(5));   // 50 — captura por referencia, no por valor
```

---

## 🧵 Extension methods

```csharp
public static class IntExtensions
{
    // "this" en el primer parámetro marca el método como extensión
    public static bool IsEven(this int number) => number % 2 == 0;
    public static int Squared(this int number) => number * number;
}

Console.WriteLine(4.IsEven());     // true
Console.WriteLine(5.Squared());    // 25

// Extension methods sobre genéricos
public static class EnumerableExtensions
{
    public static IEnumerable<T> WhereNotNull<T>(this IEnumerable<T?> source) where T : class =>
        source.Where(x => x is not null)!;
}
```

---

## 🧊 Generics

```csharp
// Clase genérica
public class Box<T>
{
    public T Value { get; set; }
    public Box(T value) => Value = value;
}

var intBox = new Box<int>(42);
var stringBox = new Box<string>("hola");

// Método genérico con restricciones (constraints)
public static T Max<T>(T a, T b) where T : IComparable<T> =>
    a.CompareTo(b) > 0 ? a : b;

// Constraints comunes
public class Repository<T> where T : class, new()
{
    public T CreateNew() => new T();
}

// class, struct, notnull, unmanaged, base class, interface, new()
public void Process<T>(T item) where T : IDisposable, new() { }

// Covarianza (out) y contravarianza (in) en interfaces genéricas
public interface IProducer<out T> { T Produce(); }
public interface IConsumer<in T> { void Consume(T item); }

IProducer<string> stringProducer = null!;
IProducer<object> objectProducer = stringProducer;   // Covarianza — válido porque T es "out"

// Generic math (C# 11+) — operadores como constraint
public static T Sum<T>(T a, T b) where T : INumber<T> => a + b;
```

---

## 🛠️ Buenas prácticas

* **`Func`/`Action`/`Predicate`** en vez de declarar `delegate` propios salvo que se necesite un nombre semántico claro.
* **Expression-bodied members (`=>`)** para métodos y propiedades de una sola expresión.
* **Extension methods** para agregar comportamiento a tipos que no controlás (tipos del BCL, DTOs generados).
* **Constraints explícitos** (`where T : ...`) en vez de `dynamic` o casts inseguros dentro de métodos genéricos.
* **Cuidado con closures sobre variables mutables** en loops — capturan la variable, no el valor en el momento de la captura.
