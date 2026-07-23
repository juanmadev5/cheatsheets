# C# — Colecciones

---

## 📌 Tabla de resumen

| Tipo | Descripción | Uso típico |
|---|---|---|
| **`T[]`** | Array de tamaño fijo | Datos de tamaño conocido, alto rendimiento |
| **`List<T>`** | Lista dinámica | Colección general de uso más común |
| **`Dictionary<TKey, TValue>`** | Mapa clave-valor | Búsquedas O(1) por clave |
| **`HashSet<T>`** | Conjunto sin duplicados | Membresía, operaciones de conjuntos |
| **`Queue<T>`** | FIFO | Procesamiento en orden de llegada |
| **`Stack<T>`** | LIFO | Deshacer, backtracking |
| **`IEnumerable<T>`** | Interfaz base iterable | Tipo de retorno más genérico posible |
| **`IReadOnlyList<T>`** | Lista de solo lectura | Exponer colecciones sin permitir mutación |

---

## 🔢 Arrays

```csharp
// Declaración e inicialización
int[] numbers = [1, 2, 3, 4, 5];              // Collection expression (C# 12+)
int[] zeros = new int[5];                      // Array de 5 ceros
string[] names = new string[] { "Ana", "Bob" };

// Multidimensional
int[,] matrix = new int[3, 3];
matrix[0, 0] = 1;

// Jagged (array de arrays)
int[][] jagged = [[1, 2], [3, 4, 5]];

// Propiedades y métodos
Console.WriteLine(numbers.Length);
Array.Sort(numbers);
Array.Reverse(numbers);
int index = Array.IndexOf(numbers, 3);

// Rangos e índices
int[] slice = numbers[1..3];
int last = numbers[^1];
```

---

## 📋 List\<T>

```csharp
List<string> names = ["Ana", "Beto", "Carla"];

names.Add("Diego");
names.AddRange(["Elena", "Fabio"]);
names.Insert(0, "Primero");
names.Remove("Beto");
names.RemoveAt(0);
names.Contains("Carla");        // true
names.IndexOf("Carla");
names.Sort();
names.Reverse();
names.Clear();

// Búsqueda
List<int> nums = [1, 2, 3, 4, 5];
int found = nums.Find(n => n > 3);          // 4 — primer match
List<int> allFound = nums.FindAll(n => n > 2);
bool any = nums.Exists(n => n > 10);        // false

// Conversión
int[] asArray = nums.ToArray();
IReadOnlyList<int> readOnly = nums.AsReadOnly();

// Capacidad — reservar espacio evita realocaciones
var buffer = new List<int>(capacity: 1000);
```

---

## 🔑 Dictionary\<TKey, TValue>

```csharp
Dictionary<string, int> ages = new()
{
    ["Ana"] = 30,
    ["Beto"] = 25
};

// Collection expression con inicializador (sintaxis clásica sigue siendo la más común)
var scores = new Dictionary<string, int> { { "Ana", 90 }, { "Beto", 85 } };

// Acceso y mutación
ages["Carla"] = 28;             // Agrega o sobreescribe
ages.Add("Diego", 40);          // Lanza si la clave ya existe
ages.TryGetValue("Ana", out int age);
bool exists = ages.ContainsKey("Ana");
ages.Remove("Beto");

// Iteración
foreach (var (name, value) in ages)
    Console.WriteLine($"{name}: {value}");

foreach (var key in ages.Keys) Console.WriteLine(key);
foreach (var value in ages.Values) Console.WriteLine(value);

// GetValueOrDefault
int defaultAge = ages.GetValueOrDefault("Nadie", 0);

// CollectionsMarshal.GetValueRefOrAddDefault — acceso avanzado sin doble lookup (.NET 6+)
```

---

## 🎯 HashSet\<T> y operaciones de conjuntos

```csharp
HashSet<int> setA = [1, 2, 3, 4];
HashSet<int> setB = [3, 4, 5, 6];

setA.Add(5);
setA.Contains(3);       // true
setA.Remove(1);

var union = new HashSet<int>(setA);
union.UnionWith(setB);              // {2,3,4,5,6}

var intersect = new HashSet<int>(setA);
intersect.IntersectWith(setB);      // {3,4,5}

var except = new HashSet<int>(setA);
except.ExceptWith(setB);            // {2}

// SortedSet<T> — mantiene orden automáticamente
SortedSet<int> sorted = [5, 1, 3, 2, 4];   // {1,2,3,4,5}
```

---

## 📥 Queue\<T> y Stack\<T>

```csharp
// Queue — FIFO
Queue<string> queue = new();
queue.Enqueue("primero");
queue.Enqueue("segundo");
string next = queue.Dequeue();     // "primero"
string peek = queue.Peek();        // "segundo" sin removerlo

// Stack — LIFO
Stack<string> stack = new();
stack.Push("uno");
stack.Push("dos");
string top = stack.Pop();          // "dos"
string peekTop = stack.Peek();     // "uno" sin removerlo
```

---

## 🔄 IEnumerable\<T> e iteradores propios

```csharp
// yield return — genera un iterador perezoso sin construir la colección completa en memoria
public IEnumerable<int> Range(int start, int count)
{
    for (int i = 0; i < count; i++)
        yield return start + i;
}

foreach (var n in Range(1, 5)) Console.Write(n);   // 12345

// yield break — corta la iteración
public IEnumerable<int> TakeUntilNegative(IEnumerable<int> source)
{
    foreach (var n in source)
    {
        if (n < 0) yield break;
        yield return n;
    }
}

// Implementar IEnumerable<T> manualmente
public class Range3 : IEnumerable<int>
{
    private readonly int _start, _count;
    public Range3(int start, int count) => (_start, _count) = (start, count);

    public IEnumerator<int> GetEnumerator()
    {
        for (int i = 0; i < _count; i++) yield return _start + i;
    }

    System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator() => GetEnumerator();
}
```

---

## 🧺 Colecciones inmutables y de solo lectura

```csharp
// System.Collections.Immutable (paquete NuGet System.Collections.Immutable)
using System.Collections.Immutable;

ImmutableList<int> immutableList = [1, 2, 3];
var updated = immutableList.Add(4);   // Nueva instancia — la original no cambia

ImmutableDictionary<string, int> immutableDict = ImmutableDictionary<string, int>.Empty
    .Add("a", 1)
    .Add("b", 2);

// Exponer colecciones internas como de solo lectura
public class Order
{
    private readonly List<string> _items = [];
    public IReadOnlyList<string> Items => _items;
}
```

---

## 🛠️ Buenas prácticas

* **`List<T>` por defecto**, `T[]` solo cuando el tamaño es fijo y conocido de antemano.
* **`IEnumerable<T>` como tipo de retorno público** cuando el llamador solo necesita iterar — oculta la implementación concreta.
* **`Dictionary<TKey, TValue>` con `TryGetValue`** en vez de `ContainsKey` + indexador — evita doble búsqueda.
* **Collection expressions `[...]`** para inicializar cualquier tipo de colección de forma uniforme.
* **`yield return`** para secuencias grandes o infinitas — evita cargar todo en memoria.
