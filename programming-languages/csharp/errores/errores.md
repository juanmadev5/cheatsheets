# C# — Manejo de Errores

---

## 📌 try / catch / finally

```csharp
try
{
    int result = 10 / int.Parse("0");
}
catch (DivideByZeroException ex)
{
    Console.WriteLine($"División por cero: {ex.Message}");
}
catch (FormatException ex) when (ex.Message.Contains("input"))   // Exception filter
{
    Console.WriteLine("Formato inválido con contexto específico");
}
catch (Exception ex)
{
    Console.WriteLine($"Error genérico: {ex.Message}");
}
finally
{
    Console.WriteLine("Se ejecuta siempre, haya o no excepción");
}

// Captura sin variable — cuando no se necesita inspeccionar la excepción
try { RiskyOperation(); }
catch (InvalidOperationException) { Console.WriteLine("Operación inválida"); }

// Re-lanzar preservando el stack trace original
try
{
    DoWork();
}
catch (Exception ex)
{
    LogError(ex);
    throw;              // Preserva el stack trace — NO usar "throw ex;"
}
```

---

## 🧱 Excepciones personalizadas

```csharp
public class InsufficientFundsException : Exception
{
    public decimal Requested { get; }
    public decimal Available { get; }

    public InsufficientFundsException(decimal requested, decimal available)
        : base($"Se pidieron {requested:C} pero solo hay {available:C} disponibles")
    {
        Requested = requested;
        Available = available;
    }

    public InsufficientFundsException(string message, Exception innerException)
        : base(message, innerException) { }
}

public class Account
{
    public decimal Balance { get; private set; }

    public void Withdraw(decimal amount)
    {
        if (amount > Balance)
            throw new InsufficientFundsException(amount, Balance);

        Balance -= amount;
    }
}

try
{
    account.Withdraw(1000);
}
catch (InsufficientFundsException ex)
{
    Console.WriteLine($"Faltan {ex.Requested - ex.Available:C}");
}
```

---

## 🎯 Excepciones comunes del BCL

| Excepción | Cuándo se lanza |
|---|---|
| **`ArgumentException`** | Argumento inválido pasado a un método |
| **`ArgumentNullException`** | Argumento `null` donde no se permite |
| **`ArgumentOutOfRangeException`** | Argumento fuera del rango permitido |
| **`InvalidOperationException`** | Llamada inválida para el estado actual del objeto |
| **`NullReferenceException`** | Acceso a un miembro de una referencia `null` (evitar con NRT) |
| **`IndexOutOfRangeException`** | Índice fuera de los límites de un array |
| **`KeyNotFoundException`** | Clave inexistente en un `Dictionary` |
| **`FormatException`** | Formato inválido al parsear (`int.Parse`, etc.) |
| **`TimeoutException`** | Una operación excede el tiempo límite |
| **`OperationCanceledException`** | Una `Task` fue cancelada vía `CancellationToken` |

```csharp
// Guard clauses idiomáticas
public void SetAge(int age)
{
    ArgumentOutOfRangeException.ThrowIfNegative(age);          // .NET 8+
    ArgumentNullException.ThrowIfNull(name);                    // .NET 6+
    ArgumentException.ThrowIfNullOrEmpty(name);                 // .NET 8+
}
```

---

## 🧹 using / IDisposable

```csharp
// using statement clásico — garantiza Dispose() al salir del bloque
using (var stream = new FileStream("data.txt", FileMode.Open))
{
    // trabajar con stream
}   // stream.Dispose() se llama automáticamente, incluso si hay excepción

// using declaration (C# 8+) — se dispone al final del scope contenedor
using var connection = new SqlConnection(connectionString);
connection.Open();

// Implementar IDisposable
public class ResourceHolder : IDisposable
{
    private bool _disposed;
    private readonly FileStream _stream = new("data.txt", FileMode.Open);

    public void Dispose()
    {
        if (_disposed) return;
        _stream.Dispose();
        _disposed = true;
        GC.SuppressFinalize(this);
    }
}

// IAsyncDisposable — para liberación asíncrona de recursos
public class AsyncResource : IAsyncDisposable
{
    public async ValueTask DisposeAsync()
    {
        await Task.Delay(10);   // Limpieza asíncrona
    }
}

await using var resource = new AsyncResource();
```

---

## 🧯 Excepciones agregadas y stack traces

```csharp
// AggregateException — común al usar Task.WhenAll con múltiples fallos
try
{
    await Task.WhenAll(task1, task2);
}
catch (Exception ex)
{
    Console.WriteLine(ex.StackTrace);
}

// InnerException — para envolver una excepción sin perder la causa original
try
{
    ParseConfig();
}
catch (FormatException ex)
{
    throw new InvalidOperationException("No se pudo cargar la configuración", ex);
}
```

---

## 🛠️ Buenas prácticas

* **`throw;` en vez de `throw ex;`** al re-lanzar — preserva el stack trace original.
* **Excepciones específicas del dominio** (`InsufficientFundsException`) en vez de `Exception` genérica.
* **No usar excepciones para control de flujo normal** — son costosas y oscurecen la intención del código.
* **`ArgumentNullException.ThrowIfNull` y similares** en vez de checks manuales repetitivos.
* **`using`/`await using`** siempre que se trabaje con `IDisposable`/`IAsyncDisposable` — evita fugas de recursos.
