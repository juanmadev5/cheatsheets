# Kotlin — Colecciones

---

## 📋 List

```kotlin
// Crear — inmutables
val empty = emptyList<Int>()
val single = listOf("solo")
val list = listOf(1, 2, 3, 4, 5)
val fromRange = (1..10).toList()
val generated = List(5) { it * 2 }   // [0, 2, 4, 6, 8]

// Crear — mutables
val mutable = mutableListOf(1, 2, 3)
val arrayList = arrayListOf("a", "b", "c")

// Acceso
list[0]; list.first(); list.last()
list.getOrNull(10)          // null si fuera de rango
list.getOrElse(10) { -1 }  // -1 si fuera de rango
list.size; list.isEmpty(); list.isNotEmpty()
list.indices                // IntRange 0..4
list.lastIndex              // 4

// Operaciones mutables
mutable.add("nuevo")
mutable.add(0, "primero")
mutable.addAll(listOf("x", "y"))
mutable.remove("a")
mutable.removeAt(0)
mutable.removeIf { it.length > 3 }
mutable.set(0, "reemplazado")
mutable[0] = "reemplazado"
mutable.clear()
mutable.sort()
mutable.sortBy { it.length }
mutable.reverse()
mutable.shuffle()

// Subconjuntos
list.subList(1, 4)          // [2, 3, 4] — view, no copia
list.slice(1..3)            // [2, 3, 4] — copia
list.slice(listOf(0, 2, 4)) // [1, 3, 5]

// Búsqueda
list.contains(3)
list.indexOf(3)
list.lastIndexOf(3)
list.binarySearch(3)        // Requiere lista ordenada

// Conversiones
list.toMutableList()
list.toSet()
list.toSortedSet()
list.reversed()
list.asReversed()           // View reversado
list.joinToString(", ")
list.joinToString(separator = ", ", prefix = "[", postfix = "]")
```

---

## 🗺️ Map

```kotlin
// Crear — inmutables
val empty = emptyMap<String, Int>()
val map = mapOf("a" to 1, "b" to 2, "c" to 3)
val sorted = sortedMapOf("c" to 3, "a" to 1, "b" to 2)   // Ordenado por clave

// Crear — mutables
val mutable = mutableMapOf("a" to 1, "b" to 2)
val hashMap = hashMapOf("a" to 1)
val linkedMap = linkedMapOf("a" to 1)   // Mantiene orden de inserción

// Acceso
map["key"]                     // null si no existe
map.getValue("key")            // Lanza NoSuchElementException si no existe
map.getOrDefault("key", 0)
map.getOrElse("key") { computeDefault() }
map.containsKey("key")
map.containsValue(42)
map.size; map.isEmpty(); map.isNotEmpty()

// Iteración
for ((key, value) in map) println("$key: $value")
map.forEach { (key, value) -> println("$key: $value") }
map.keys; map.values; map.entries

// Operaciones mutables
mutable["newKey"] = 100
mutable.put("key", 200)
mutable.putIfAbsent("key", 300)   // Solo si no existe
mutable.getOrPut("key") { computeValue() }  // Obtener o insertar
mutable.remove("key")
mutable.remove("key", 42)         // Remove solo si el valor coincide

// Transformaciones
map.mapKeys { (k, _) -> k.uppercase() }
map.mapValues { (_, v) -> v * 2 }
map.filter { (_, v) -> v > 1 }
map.filterKeys { it.startsWith("a") }
map.filterValues { it > 0 }
map.any { (_, v) -> v > 5 }
map.all { (_, v) -> v > 0 }
map.none { (_, v) -> v < 0 }
map.count { (_, v) -> v.isEven() }
map.maxBy { (_, v) -> v }
map.minByOrNull { (_, v) -> v }

// Merge / combine
val merged = map + ("d" to 4)        // Nuevo mapa con "d"
val merged2 = map + mapOf("d" to 4, "e" to 5)
val without = map - "a"              // Nuevo mapa sin "a"
val withoutMultiple = map - setOf("a", "b")

// Conversiones
map.toSortedMap()
map.toList()                         // List<Pair<K,V>>
map.entries.toList()
```

---

## 🔢 Set

```kotlin
// Crear — inmutables
val empty = emptySet<Int>()
val set = setOf(1, 2, 3, 4, 5)
val sorted = sortedSetOf(3, 1, 4, 1, 5)   // {1, 3, 4, 5} (sin duplicados, ordenado)

// Crear — mutables
val mutable = mutableSetOf(1, 2, 3)
val hashSet = hashSetOf(1, 2, 3)
val linkedSet = linkedSetOf(3, 1, 2)   // Mantiene orden de inserción

// Operaciones
mutable.add(4)
mutable.addAll(listOf(5, 6, 7))
mutable.remove(1)
mutable.removeIf { it > 5 }
mutable.contains(3)
mutable.size; mutable.isEmpty()

// Operaciones de conjuntos
val a = setOf(1, 2, 3, 4)
val b = setOf(3, 4, 5, 6)

val union = a + b              // {1, 2, 3, 4, 5, 6} — unión
val intersect = a intersect b  // {3, 4}             — intersección
val subtract = a subtract b    // {1, 2}             — diferencia
val sym = (a - b) + (b - a)   // {1, 2, 5, 6}       — diferencia simétrica

a.containsAll(setOf(1, 2))     // true — subconjunto

// Conversiones
set.toMutableSet()
set.toList()
set.sorted()
```

---

## 🌊 Sequence — evaluación lazy

```kotlin
// Sequence — procesa elemento a elemento (lazy), ideal para cadenas largas
// List — procesa toda la colección en cada paso (eager)

// Crear sequences
val seq1 = sequenceOf(1, 2, 3, 4, 5)
val seq2 = (1..1_000_000).asSequence()   // Lazy — no crea un millón de enteros
val seq3 = generateSequence(1) { it + 1 }  // Infinita: 1, 2, 3, 4...
val seq4 = generateSequence(seed = 1) { if (it < 100) it * 2 else null }  // Finita
val seq5 = sequence {
    yield(1)
    yield(2)
    yieldAll(listOf(3, 4, 5))
    yieldAll(generateSequence(6) { if (it < 10) it + 1 else null })
}

// Procesamiento lazy — solo se evalúa hasta el final con terminal op
val result = (1..1_000_000).asSequence()
    .filter { it % 2 == 0 }         // No evalúa aún
    .map { it * it }                 // No evalúa aún
    .take(5)                         // No evalúa aún
    .toList()                        // ← Terminal: AQUÍ se evalúa
// Solo procesa los primeros 5 elementos que pasen el filtro

// Sequence infinita con takeWhile
val primes = generateSequence(2) { n ->
    (n + 1..Int.MAX_VALUE).first { candidate ->
        (2 until candidate).none { candidate % it == 0 }
    }
}
primes.take(10).toList()   // [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]

// ¿Cuándo usar Sequence vs List?
// List: < 1000 elementos, operaciones simples
// Sequence: > 1000 elementos, muchas operaciones encadenadas,
//           colecciones potencialmente infinitas

// Fibonacci con sequence
val fibonacci = sequence {
    var a = 0L; var b = 1L
    while (true) {
        yield(a)
        val next = a + b; a = b; b = next
    }
}
fibonacci.take(10).toList()  // [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

---

## 🏗️ Builders de colecciones

```kotlin
// buildList — construcción con DSL
val items = buildList {
    add("primero")
    addAll(listOf("segundo", "tercero"))
    if (condition) add("condicional")
    repeat(3) { add("item_$it") }
}

// buildMap
val config = buildMap<String, Any> {
    put("host", "localhost")
    put("port", 8080)
    put("debug", true)
    putAll(extraConfig)
    if (isProduction) put("ssl", true)
}

// buildSet
val uniqueIds = buildSet {
    addAll(existingIds)
    add(newId)
    removeAll(deletedIds)
}

// buildString — StringBuilder con DSL
val message = buildString {
    append("Hola ")
    append(name)
    appendLine()
    repeat(3) { append("item_$it\n") }
    insert(0, "HEADER\n")
}

// Sequence builder
val customSeq = sequence {
    yield(1)
    for (i in 2..5) {
        if (i.isEven) yield(i * 10)
        else yieldAll(listOf(i, i * 2))
    }
}
```

---

## 🎁 Destructuring, Pair, Triple

```kotlin
// Pair
val pair = Pair("Hola", 42)
val pair2 = "Hola" to 42    // Forma idiomática
val (str, num) = pair        // Destructuring

// Triple
val triple = Triple("a", 1, true)
val (a, b, c) = triple

// Destructuring de data class
data class Point(val x: Double, val y: Double)
val point = Point(3.0, 4.0)
val (x, y) = point

// _ para ignorar componentes
val (_, second, _) = triple
val (id, _, email) = user    // Ignorar nombre

// Destructuring en lambdas
val map = mapOf("a" to 1, "b" to 2)
map.forEach { (key, value) -> println("$key=$value") }

val pairs = listOf(1 to "uno", 2 to "dos")
pairs.map { (num, word) -> "$num: $word" }

// componentN() — implementar destructuring en clases propias
class Color(val r: Int, val g: Int, val b: Int) {
    operator fun component1() = r
    operator fun component2() = g
    operator fun component3() = b
}
val color = Color(255, 128, 0)
val (red, green, blue) = color

// Destructuring en for
for ((index, value) in list.withIndex()) { ... }
for ((key, value) in map) { ... }
for ((first, second) in listOfPairs) { ... }

// Retornar múltiples valores — usar data class o Pair/Triple
fun divmod(a: Int, b: Int): Pair<Int, Int> = a / b to a % b
val (quotient, remainder) = divmod(17, 5)

// Con data class (más expresivo)
data class DivResult(val quotient: Int, val remainder: Int)
fun divmod2(a: Int, b: Int) = DivResult(a / b, a % b)
val (q, r) = divmod2(17, 5)
```
