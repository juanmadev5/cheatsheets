# C# — LINQ

---

## 📌 Method syntax vs Query syntax

LINQ (Language Integrated Query) permite consultar cualquier `IEnumerable<T>` (en memoria) o `IQueryable<T>` (traducido a SQL por un provider como EF Core) con una sintaxis declarativa.

```csharp
List<int> numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Method syntax — la más usada en la práctica
var evensMethod = numbers.Where(n => n % 2 == 0).Select(n => n * n);

// Query syntax — más cercana a SQL, se traduce a method syntax en compilación
var evensQuery =
    from n in numbers
    where n % 2 == 0
    select n * n;
```

---

## 🔍 Filtrado y proyección

```csharp
List<Person> people =
[
    new("Ana", 30, "Buenos Aires"),
    new("Beto", 17, "Córdoba"),
    new("Carla", 45, "Buenos Aires"),
];

// Where — filtrar
var adults = people.Where(p => p.Age >= 18);

// Select — proyectar
var names = people.Select(p => p.Name);
var summaries = people.Select(p => $"{p.Name} ({p.Age})");

// Select con índice
var indexed = people.Select((p, i) => $"{i}: {p.Name}");

// SelectMany — aplanar colecciones anidadas
List<List<int>> nested = [[1, 2], [3, 4], [5]];
var flat = nested.SelectMany(x => x);   // [1,2,3,4,5]

// OfType — filtrar por tipo
List<object> mixed = [1, "dos", 3, "cuatro"];
var onlyInts = mixed.OfType<int>();
```

---

## 🔀 Ordenamiento y agrupación

```csharp
// OrderBy / OrderByDescending / ThenBy
var sorted = people.OrderBy(p => p.City).ThenByDescending(p => p.Age);

// GroupBy
var byCity = people.GroupBy(p => p.City);
foreach (var group in byCity)
{
    Console.WriteLine($"{group.Key}: {group.Count()} personas");
    foreach (var person in group) Console.WriteLine($"  {person.Name}");
}

// GroupBy con proyección de resultado
var citySummary = people
    .GroupBy(p => p.City)
    .Select(g => new { City = g.Key, AvgAge = g.Average(p => p.Age) });
```

---

## 🔗 Joins

```csharp
List<Order> orders = [new(1, "Ana"), new(2, "Beto")];
List<Product> products = [new(1, "Laptop", 1), new(2, "Mouse", 1), new(3, "Teclado", 2)];

// Join — inner join clásico
var result = orders.Join(
    products,
    order => order.Id,
    product => product.OrderId,
    (order, product) => new { order.Customer, product.Name });

// GroupJoin — left join (grupo vacío si no hay match)
var grouped = orders.GroupJoin(
    products,
    order => order.Id,
    product => product.OrderId,
    (order, matchedProducts) => new { order.Customer, Products = matchedProducts });
```

---

## 🧮 Agregación

```csharp
List<int> nums = [1, 2, 3, 4, 5];

nums.Count();                  // 5
nums.Count(n => n > 2);        // 3
nums.Sum();                    // 15
nums.Average();                // 3.0
nums.Min();                    // 1
nums.Max();                    // 5
nums.Aggregate((a, b) => a + b);              // 15 — reduce sin seed
nums.Aggregate(100, (acc, n) => acc + n);     // 115 — con seed

// Any / All
nums.Any(n => n > 4);          // true
nums.All(n => n > 0);          // true
nums.Any();                    // true si tiene al menos un elemento

// First / Single / Last y sus variantes seguras
nums.First();                          // Lanza si está vacío
nums.FirstOrDefault();                 // 0 (default de int) si está vacío
nums.FirstOrDefault(n => n > 10, -1);  // -1 — con valor por defecto explícito (.NET 6+)
nums.Single(n => n == 3);              // Lanza si hay 0 o más de 1 match
nums.SingleOrDefault(n => n == 99);    // null/default si no hay match
nums.Last();
nums.ElementAt(2);
```

---

## 🧰 Conjuntos y transformación

```csharp
List<int> a = [1, 2, 3, 4];
List<int> b = [3, 4, 5, 6];

a.Distinct();                  // Elimina duplicados
a.Union(b);                    // {1,2,3,4,5,6}
a.Intersect(b);                // {3,4}
a.Except(b);                   // {1,2}
a.Concat(b);                   // [1,2,3,4,3,4,5,6] — sin eliminar duplicados
a.Zip(b, (x, y) => x + y);     // [4,6,8,10] — combina por posición

a.Skip(2);                     // Salta los primeros 2
a.Take(2);                     // Toma los primeros 2
a.SkipWhile(n => n < 3);
a.TakeWhile(n => n < 3);
a.Chunk(2);                    // Divide en grupos de tamaño 2 (.NET 6+)
a.Reverse();
a.ToList();
a.ToDictionary(n => n, n => n * n);
a.ToHashSet();
```

---

## ⚡ Ejecución diferida y materialización

```csharp
// LINQ es lazy — no se ejecuta hasta que se enumera
var query = numbers.Where(n => n > 5);   // Todavía no filtró nada

foreach (var n in query) Console.WriteLine(n);   // Acá se ejecuta

// ToList/ToArray fuerzan la ejecución inmediata (materialización)
var materialized = numbers.Where(n => n > 5).ToList();

// Cuidado: reevaluar una query lazy sobre una fuente mutable puede dar resultados distintos
var list = new List<int> { 1, 2, 3 };
var lazyQuery = list.Where(n => n > 1);
list.Add(4);
// lazyQuery ahora incluye el 4 al enumerarse, porque se re-ejecuta sobre la lista actual
```

---

## 🛠️ Buenas prácticas

* **Method syntax** para la mayoría de los casos — más composable y familiar.
* **`FirstOrDefault`/`SingleOrDefault`** en vez de `First`/`Single` cuando el resultado puede no existir.
* **Materializar con `.ToList()`** antes de iterar varias veces sobre el mismo resultado, para evitar recalcular la query.
* **Evitar LINQ sobre `IQueryable<T>` (EF Core) con lógica no traducible a SQL** — falla en runtime o trae todo a memoria.
* **`Any()` en vez de `Count() > 0`** para chequear si una colección tiene elementos — más eficiente.
