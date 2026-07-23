# C# — Async / await

---

## 📌 Task y async/await

```csharp
// Método async básico — Task representa una operación sin valor de retorno
public async Task GreetAsync()
{
    await Task.Delay(1000);
    Console.WriteLine("Hola después de 1 segundo");
}

// Task<T> — operación que retorna un valor
public async Task<string> FetchDataAsync()
{
    using var client = new HttpClient();
    return await client.GetStringAsync("https://api.example.com/data");
}

// Llamada y espera
string data = await FetchDataAsync();

// ValueTask<T> — evita allocation cuando el resultado suele estar disponible sincrónicamente
public async ValueTask<int> GetCachedOrComputeAsync(int key)
{
    if (_cache.TryGetValue(key, out int value)) return value;   // Camino síncrono, sin Task real
    return await ComputeAsync(key);
}
```

---

## 🔀 Ejecución concurrente

```csharp
// Task.WhenAll — espera a que todas terminen, en paralelo
Task<string> t1 = FetchDataAsync("url1");
Task<string> t2 = FetchDataAsync("url2");
Task<string> t3 = FetchDataAsync("url3");

string[] results = await Task.WhenAll(t1, t2, t3);

// Task.WhenAny — continúa apenas la primera termine
Task<string> winner = await Task.WhenAny(t1, t2, t3);
string firstResult = await winner;

// Task.Run — ejecutar trabajo CPU-bound en el thread pool
int result = await Task.Run(() => ExpensiveComputation());

// Combinando WhenAll con LINQ para N tareas dinámicas
var urls = new[] { "url1", "url2", "url3" };
var tasks = urls.Select(FetchDataAsync);
string[] allResults = await Task.WhenAll(tasks);
```

---

## ⏱️ CancellationToken

```csharp
public async Task<string> FetchWithTimeoutAsync(CancellationToken cancellationToken)
{
    using var client = new HttpClient();
    return await client.GetStringAsync("https://api.example.com/data", cancellationToken);
}

// Crear un token con timeout
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(5));
try
{
    string data = await FetchWithTimeoutAsync(cts.Token);
}
catch (OperationCanceledException)
{
    Console.WriteLine("Operación cancelada por timeout");
}

// Cancelación manual y linked tokens
using var manualCts = new CancellationTokenSource();
using var linked = CancellationTokenSource.CreateLinkedTokenSource(manualCts.Token, cts.Token);
manualCts.Cancel();

// Chequeo manual dentro de loops largos
public void ProcessItems(IEnumerable<int> items, CancellationToken token)
{
    foreach (var item in items)
    {
        token.ThrowIfCancellationRequested();
        // procesar item...
    }
}
```

---

## 🔁 IAsyncEnumerable\<T>

```csharp
// Streams asíncronos — combina yield return con await (C# 8+)
public async IAsyncEnumerable<int> GenerateNumbersAsync(
    [System.Runtime.CompilerServices.EnumeratorCancellation] CancellationToken token = default)
{
    for (int i = 0; i < 10; i++)
    {
        await Task.Delay(100, token);
        yield return i;
    }
}

// Consumo con await foreach
await foreach (var number in GenerateNumbersAsync())
{
    Console.WriteLine(number);
}

// Con cancelación
await foreach (var number in GenerateNumbersAsync().WithCancellation(cts.Token))
{
    Console.WriteLine(number);
}
```

---

## 🛡️ Manejo de excepciones en código async

```csharp
public async Task<string> SafeFetchAsync()
{
    try
    {
        return await FetchDataAsync();
    }
    catch (HttpRequestException ex)
    {
        Console.WriteLine($"Error de red: {ex.Message}");
        return string.Empty;
    }
}

// Task.WhenAll agrega todas las excepciones en un AggregateException al hacer .Wait(),
// pero await Task.WhenAll(...) relanza solo la primera — inspeccionar cada Task si hace falta el resto
var tasks = new[] { FailingTaskAsync(), FailingTaskAsync() };
try
{
    await Task.WhenAll(tasks);
}
catch
{
    var errors = tasks.Where(t => t.IsFaulted).SelectMany(t => t.Exception!.InnerExceptions);
    foreach (var error in errors) Console.WriteLine(error.Message);
}
```

---

## 🧵 ConfigureAwait y contexto de sincronización

```csharp
// ConfigureAwait(false) — evita volver al SynchronizationContext original
// Recomendado en librerías; innecesario en la mayoría de apps ASP.NET Core modernas (sin SynchronizationContext)
public async Task<string> LibraryMethodAsync()
{
    return await FetchDataAsync().ConfigureAwait(false);
}
```

---

## 🛠️ Buenas prácticas

* **`async`/`await` de punta a punta** — evitar mezclar con `.Result`/`.Wait()`, que puede causar deadlocks.
* **`Task.WhenAll`** para operaciones independientes en paralelo, en vez de `await` secuenciales.
* **Nombrar métodos async con sufijo `Async`** — convención del BCL y muy usada en la comunidad.
* **Propagar `CancellationToken`** en toda la cadena de llamadas async de una operación cancelable.
* **`ValueTask<T>`** solo cuando el perfil de uso lo justifica (alto volumen, resultado frecuentemente síncrono) — en el resto de los casos, `Task<T>` es más simple y seguro.
