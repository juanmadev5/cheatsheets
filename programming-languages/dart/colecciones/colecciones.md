# Dart — Colecciones

---

## 📋 Listas

```dart
// Crear
List<int> empty = [];
List<int> fixed = List.filled(5, 0);           // [0, 0, 0, 0, 0]
List<int> generated = List.generate(5, (i) => i * 2);  // [0,2,4,6,8]
List<int> unmod = List.unmodifiable([1, 2, 3]);
var literal = [1, 2, 3, 4, 5];

// Acceso
list[0]; list.last; list.first;
list.length; list.isEmpty; list.isNotEmpty;

// Agregar / quitar
list.add(6);
list.addAll([7, 8, 9]);
list.insert(0, 0);
list.insertAll(0, [-2, -1]);
list.remove(3);            // Eliminar primera ocurrencia del valor
list.removeAt(0);
list.removeLast();
list.removeWhere((e) => e < 0);
list.clear();

// Búsqueda
list.contains(3);
list.indexOf(3);           // -1 si no existe
list.lastIndexOf(3);
list.indexWhere((e) => e > 5);
list.firstWhere((e) => e > 5, orElse: () => -1);
list.singleWhere((e) => e == 3);

// Ordenar
list.sort();
list.sort((a, b) => b.compareTo(a));   // Descendente
final sorted = [...list]..sort();      // Copia ordenada

// Sub-listas
list.sublist(1, 4);
list.take(3).toList();
list.skip(2).toList();
list.takeWhile((e) => e < 5).toList();
list.skipWhile((e) => e < 5).toList();

// Transformaciones
list.map((e) => e * 2).toList();
list.where((e) => e.isEven).toList();
list.expand((e) => [e, e * 2]).toList();
list.reduce((a, b) => a + b);
list.fold(0, (acc, e) => acc + e);
list.reversed.toList();
list.toSet();    // Elimina duplicados

// Iterar con índice
list.asMap().forEach((index, value) => print("$index: $value"));
list.indexed.forEach((p) => print("${p.$1}: ${p.$2}"));  // Dart 3.x

// Chequeos
list.any((e) => e > 5);
list.every((e) => e > 0);

// Join
["a", "b", "c"].join(", ");   // "a, b, c"
```

---

## 🗺️ Mapas

```dart
// Crear
Map<String, int> empty = {};
Map<String, int> literal = {"a": 1, "b": 2};
Map<String, int> fromEntries = Map.fromEntries([
  MapEntry("a", 1), MapEntry("b", 2),
]);
Map<String, int> fromIterables = Map.fromIterables(
  ["a", "b"], [1, 2],
);
final unmod = Map.unmodifiable(literal);

// Acceso
map["key"];                      // null si no existe
map["key"]!;                     // Lanza si null
map.containsKey("key");
map.containsValue(42);

// Agregar / modificar
map["newKey"] = 100;
map.putIfAbsent("key", () => 0);
map.update("key", (v) => v + 1);
map.update("key", (v) => v + 1, ifAbsent: () => 1);
map.addAll({"x": 10, "y": 20});

// Eliminar
map.remove("key");
map.removeWhere((k, v) => v < 0);
map.clear();

// Iterar
map.forEach((key, value) => print("$key: $value"));
for (final MapEntry(:key, :value) in map.entries) {
  print("$key: $value");
}
map.keys;      // Iterable<String>
map.values;    // Iterable<int>
map.entries;   // Iterable<MapEntry<String, int>>

// Transformar
map.map((k, v) => MapEntry(k.toUpperCase(), v * 2));

// Cast
final typed = (rawDynamic as Map).cast<String, int>();

// Null-aware
int safe = map["key"] ?? 0;
```

---

## 🔢 Sets

```dart
// Crear
Set<int> empty = {};
Set<int> literal = {1, 2, 3};
Set<int> from = Set.from([1, 2, 2, 3]);   // {1, 2, 3}
Set<int> unmod = Set.unmodifiable({1, 2, 3});

// Operaciones básicas
set.add(4);
set.addAll([5, 6, 7]);
set.remove(1);
set.contains(3);

// Operaciones de conjuntos
final a = {1, 2, 3, 4};
final b = {3, 4, 5, 6};

a.union(b);          // {1, 2, 3, 4, 5, 6}
a.intersection(b);   // {3, 4}
a.difference(b);     // {1, 2}

// Conversión
set.toList();
list.toSet();   // Elimina duplicados
```

---

## 🔁 Operaciones funcionales

```dart
final numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// map — transformar
final doubled = numbers.map((n) => n * 2).toList();

// where — filtrar
final evens = numbers.where((n) => n.isEven).toList();

// reduce — acumular (lanza si vacío)
final sum = numbers.reduce((acc, n) => acc + n);        // 55

// fold — con valor inicial (no lanza)
final sum2 = numbers.fold<int>(0, (acc, n) => acc + n); // 55
final product = numbers.fold<int>(1, (acc, n) => acc * n);

// expand — flatMap
final nested = [[1, 2], [3, 4], [5, 6]];
final flat = nested.expand((list) => list).toList();     // [1,2,3,4,5,6]

// any / every
numbers.any((n) => n > 9);       // true
numbers.every((n) => n > 0);     // true

// take / skip
numbers.take(3).toList();                       // [1, 2, 3]
numbers.skip(7).toList();                       // [8, 9, 10]
numbers.takeWhile((n) => n < 5).toList();       // [1, 2, 3, 4]
numbers.skipWhile((n) => n < 5).toList();       // [5, 6, 7, 8, 9, 10]

// Encadenamiento — lazy (no evalúa hasta iterar)
final result = numbers
    .where((n) => n.isEven)
    .map((n) => n * n)
    .where((n) => n > 10)
    .take(3)
    .toList();    // [16, 36, 64]

// zip con indexed (no existe built-in zip)
final a = [1, 2, 3];
final b = ["a", "b", "c"];
final zipped = a.indexed.map((e) => (e.$2, b[e.$1])).toList();
// [(1,"a"), (2,"b"), (3,"c")]
```

---

## 🌿 Spread, collection if, collection for

```dart
// Spread operator ...
final combined = [...list1, ...list2];
final withExtra = [0, ...list1, ...list2, 7];

// Null-aware spread ...?
List<int>? nullable;
final safe = [...?nullable, 1, 2, 3];    // [1, 2, 3]

// Map spread — el último gana en colisiones
final merged = {...defaults, ...userPrefs};

// Set spread
final union = {...set1, ...set2};

// Collection if
final menu = [
  "Inicio",
  if (isLoggedIn) "Perfil",
  if (isLoggedIn) "Configuración",
  if (!isLoggedIn) "Iniciar sesión",
];

// Collection if con else y spread
final items = [
  "Gratis",
  if (isPro) ...[
    "Feature Premium 1",
    "Feature Premium 2",
  ] else
    "Actualizar a Pro",
];

// Collection for
final squares = [for (int i = 1; i <= 5; i++) i * i];   // [1,4,9,16,25]

final users = [
  for (final name in ["Ana", "Juan", "María"]) User(name: name),
];

// Combinación
final activeItems = [
  for (final item in allItems)
    if (item.isActive) item.name,
];

// En mapas
final colorMap = {
  for (final color in colors) color.name: color.hex,
  if (includeClear) "clear": "#00000000",
};
```
