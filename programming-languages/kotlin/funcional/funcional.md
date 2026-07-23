# Kotlin — Programación funcional

---

## 🔮 Lambdas y function types

```kotlin
// Function type
val square: (Int) -> Int = { x -> x * x }
val add: (Int, Int) -> Int = { a, b -> a + b }
val greet: (String) -> Unit = { name -> println("Hola $name") }
val noArgs: () -> String = { "valor" }

// it — nombre implícito cuando hay un solo parámetro
val double: (Int) -> Int = { it * 2 }
val isEven: (Int) -> Boolean = { it % 2 == 0 }

// Lambda con múltiples líneas — el último valor es el retorno
val process: (String) -> String = { input ->
    val trimmed = input.trim()
    val upper = trimmed.uppercase()
    upper  // retorno implícito
}

// Tipos que aceptan nullables
val nullableFn: ((Int) -> Int)? = null
nullableFn?.invoke(5)   // null si nullableFn es null

// Función con lambda como último parámetro — trailing lambda
fun buildString(action: StringBuilder.() -> Unit): String {
    val sb = StringBuilder()
    sb.action()   // action se llama en el contexto de StringBuilder
    return sb.toString()
}

val result = buildString {
    append("Hola")
    append(", ")
    append("mundo")
}

// Function types con receiver
val greetFn: String.() -> String = { "Hola $this" }
"Juan".greetFn()   // "Hola Juan"

// Pasar función existente como referencia
fun double(x: Int) = x * 2
val fn = ::double                     // Referencia a función top-level
val upperFn = String::uppercase       // Referencia a método
val lengthFn = String::length         // Referencia a propiedad
val constructorFn = ::User            // Referencia al constructor

listOf("a", "bb", "ccc").map(String::length)  // [1, 2, 3]
listOf("hola", "mundo").map(String::uppercase)

// Lambdas que capturan variables (closures)
fun makeCounter(): () -> Int {
    var count = 0
    return { ++count }
}
val counter = makeCounter()
counter()   // 1
counter()   // 2
counter()   // 3

// Partial application — currying manual
fun multiply(a: Int, b: Int) = a * b
val triple = { b: Int -> multiply(3, b) }
triple(5)   // 15

// Lambda con return explícito usando label
val list = listOf(1, 2, 3, 4, 5)
list.forEach { item ->
    if (item == 3) return@forEach   // Continuar al siguiente (como continue)
    println(item)
}
```

---

## 📌 inline, noinline, crossinline

```kotlin
// inline — el compilador copia el cuerpo de la función en el lugar de la llamada
// Elimina el overhead de crear objetos lambda
inline fun <T> measureTime(block: () -> T): Pair<T, Long> {
    val start = System.currentTimeMillis()
    val result = block()
    return result to System.currentTimeMillis() - start
}

// reified — acceder al tipo genérico en tiempo de ejecución
// Solo disponible en funciones inline
inline fun <reified T> List<*>.filterIsInstance(): List<T> =
    filter { it is T }.map { it as T }

inline fun <reified T> String.fromJson(): T =
    objectMapper.readValue(this, T::class.java)

inline fun <reified T> isOfType(value: Any): Boolean = value is T

// Uso de reified
val strings = listOf(1, "a", 2, "b", 3).filterIsInstance<String>()  // ["a", "b"]
isOfType<String>("hola")   // true — sin reified necesitaría Class<T> como parámetro

// noinline — excluir un parámetro lambda de ser inline
inline fun runParallel(
    task1: () -> Unit,
    noinline task2: () -> Unit    // noinline → se puede pasar como objeto
) {
    thread { task1() }
    val stored = task2            // ✅ Se puede almacenar porque es noinline
    thread { stored() }
}

// crossinline — lambda no puede tener return no-local
inline fun asyncRun(crossinline block: () -> Unit) {
    thread {
        block()   // crossinline → no permite return que salte de asyncRun
    }
}

asyncRun {
    println("en otro thread")
    // return  // ❌ Error: no se permite return no-local aquí
}

// return no-local — solo posible con inline
inline fun findFirst(list: List<Int>, predicate: (Int) -> Boolean): Int? {
    for (item in list) {
        if (predicate(item)) return item  // return de findFirst — posible por inline
    }
    return null
}
```

---

## 🎯 Scope functions

```kotlin
// let — transformar y operar sobre el resultado; null-safety
val result = "hola".let { str ->
    println("Procesando: $str")
    str.uppercase()   // retorno del let
}   // result = "HOLA"

// Con nullable — el bloque solo se ejecuta si no es null
val email: String? = getEmail()
val domain = email?.let { it.substringAfter("@") }

// Encadenado
val processed = "  Hola Mundo  "
    .let { it.trim() }
    .let { it.lowercase() }
    .let { it.replace(" ", "_") }
// "hola_mundo"

// run — ejecutar bloque en contexto del objeto (this), retorna el resultado del bloque
val length = "hola mundo".run {
    println("Procesando: $this")
    length   // this.length
}

// run sin receiver — bloque de código con resultado
val connection = run {
    val host = config.host
    val port = config.port
    createConnection(host, port)
}

// with — como run pero el objeto es el primer argumento (no extension)
val message = with(StringBuilder()) {
    append("Hola")
    append(", ")
    append("Mundo")
    toString()   // retorno — this.toString()
}

// apply — ejecutar bloque de configuración, retorna el objeto mismo (this)
val person = Person("Juan", 28).apply {
    email = "juan@example.com"
    score = 100
    role = Role.ADMIN
}   // person es el Person configurado

// Ideal para builders y configuración
val dialog = AlertDialog.Builder(context).apply {
    setTitle("Confirmar")
    setMessage("¿Estás seguro?")
    setPositiveButton("Sí") { _, _ -> confirm() }
    setNegativeButton("No") { _, _ -> cancel() }
}.create()

// also — ejecutar efecto secundario, retorna el objeto mismo (it)
val list = mutableListOf<String>()
    .also { println("Lista creada: $it") }

val user = createUser().also {
    logger.info("Usuario creado: ${it.id}")
    auditService.log("USER_CREATED", it.id)
}   // user es el User creado, el log es un efecto secundario

// Resumen rápido:
// let   → it,   retorna el resultado del bloque — transformar/null-safe
// run   → this, retorna el resultado del bloque — operar y obtener valor
// with  → this, retorna el resultado del bloque — agrupar operaciones
// apply → this, retorna el objeto             — configurar/builder
// also  → it,   retorna el objeto             — efectos secundarios/debug

// takeIf / takeUnless — filtrar con condición
val validName = name.takeIf { it.isNotBlank() }   // null si está en blanco
val nonEmpty = list.takeUnless { it.isEmpty() }    // null si está vacía

val userId = input.trim()
    .takeIf { it.matches(Regex("\\d+")) }
    ?.toLong()
    ?: throw IllegalArgumentException("ID inválido: $input")
```

---

## 🔁 Funciones de orden superior

```kotlin
val numbers = listOf(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

// map — transformar cada elemento
val doubled = numbers.map { it * 2 }           // [2, 4, 6, 8, 10, ...]
val strings = numbers.map { "item_$it" }

// filter / filterNot
val evens = numbers.filter { it % 2 == 0 }     // [2, 4, 6, 8, 10]
val odds = numbers.filterNot { it % 2 == 0 }   // [1, 3, 5, 7, 9]

// reduce — acumular (lanza si vacío)
val sum = numbers.reduce { acc, n -> acc + n }  // 55

// fold — con valor inicial (no lanza si vacío)
val sum2 = numbers.fold(0) { acc, n -> acc + n }
val product = numbers.fold(1L) { acc, n -> acc * n }

// flatMap — map + flatten
val nested = listOf(listOf(1, 2), listOf(3, 4), listOf(5, 6))
val flat = nested.flatMap { it }                // [1, 2, 3, 4, 5, 6]
val words = listOf("Hola mundo", "foo bar").flatMap { it.split(" ") }

// flatten — aplanar lista de listas
val flattened = nested.flatten()

// any / all / none
numbers.any { it > 9 }     // true
numbers.all { it > 0 }     // true
numbers.none { it > 10 }   // true

// count con predicado
numbers.count { it.isEven() }   // 5

// find / firstOrNull / lastOrNull
numbers.find { it > 5 }          // 6
numbers.firstOrNull { it > 100 } // null
numbers.lastOrNull { it < 5 }    // 4
numbers.single { it == 5 }       // 5 (lanza si hay más de uno)
numbers.singleOrNull { it > 9 }  // 10

// take / drop
numbers.take(3)                  // [1, 2, 3]
numbers.drop(7)                  // [8, 9, 10]
numbers.takeWhile { it < 5 }    // [1, 2, 3, 4]
numbers.dropWhile { it < 5 }    // [5, 6, 7, 8, 9, 10]
numbers.takeLast(3)              // [8, 9, 10]
numbers.dropLast(3)             // [1, 2, 3, 4, 5, 6, 7]

// sorted
numbers.sorted()
numbers.sortedDescending()
numbers.sortedBy { it % 3 }     // Por módulo 3
numbers.sortedByDescending { it.toString().length }

// group / partition
val (evens2, odds2) = numbers.partition { it % 2 == 0 }
val byParity = numbers.groupBy { if (it % 2 == 0) "par" else "impar" }
// {"par": [2,4,6,8,10], "impar": [1,3,5,7,9]}

val grouped = numbers.groupBy { it % 3 }   // {1:[1,4,7,10], 2:[2,5,8], 0:[3,6,9]}

// associate / associateBy / associateWith
val products = listOf(Product(1L, "Laptop", 999.99, "tech"))
val byId = products.associateBy { it.id }          // Map<Long, Product>
val nameById = products.associate { it.id to it.name }  // Map<Long, String>
val priceByName = products.associateWith { it.price }   // Map<Product, Double>

// zip — combinar dos listas
val names = listOf("Ana", "Juan", "María")
val ages = listOf(25, 30, 28)
val zipped = names.zip(ages)              // [(Ana,25), (Juan,30), (María,28)]
val mapped = names.zip(ages) { name, age -> "$name:$age" }

// unzip
val (nameList, ageList) = zipped.unzip()

// windowed / chunked
numbers.chunked(3)                // [[1,2,3],[4,5,6],[7,8,9],[10]]
numbers.windowed(3)               // [[1,2,3],[2,3,4],[3,4,5]...]
numbers.windowed(3, step = 2)     // [[1,2,3],[3,4,5],[5,6,7],[7,8,9]]

// distinct / distinctBy
listOf(1, 2, 2, 3, 3, 3).distinct()          // [1, 2, 3]
products.distinctBy { it.category }

// min / max / sum / average
numbers.min()
numbers.max()
numbers.sum()
numbers.average()
numbers.minBy { it % 3 }
numbers.maxByOrNull { it.toString().length }
numbers.sumOf { it.toLong() }
products.sumOf { it.price }

// onEach — efecto secundario sin transformar (como forEach pero retorna la colección)
val result = numbers.onEach { println(it) }.filter { it > 5 }

// scan — como fold pero retorna cada paso intermedio
numbers.scan(0) { acc, n -> acc + n }  // [0, 1, 3, 6, 10, 15, 21, 28, 36, 45, 55]

// runningFold / runningReduce
numbers.runningReduce { acc, n -> acc + n }  // [1, 3, 6, 10, 15, ...]
```

---

## 🧮 Generics

```kotlin
// Clase genérica
class Box<T>(val value: T) {
    fun <R> transform(fn: (T) -> R): Box<R> = Box(fn(value))
    override fun toString() = "Box($value)"
}

// Bounds — restricción de tipo
class NumberBox<T : Number>(val value: T) {
    fun doubled(): Double = value.toDouble() * 2
    fun isPositive() = value.toDouble() > 0
}

// Múltiples bounds con where
fun <T> maxOf(a: T, b: T): T where T : Comparable<T>, T : Number =
    if (a > b) a else b

// out — covariante (Producer) — solo produce T
// Si Box<out T>, Box<Cat> es subtipo de Box<Animal>
interface Producer<out T> {
    fun produce(): T
}

// in — contravariante (Consumer) — solo consume T
// Si Box<in T>, Box<Animal> es subtipo de Box<Cat>
interface Consumer<in T> {
    fun consume(item: T)
}

// Invariante — solo acepta exactamente T
class MutableBox<T>(var value: T)

// Star projection — cuando no importa el tipo
fun printList(list: List<*>) {
    list.forEach { println(it) }
}
fun getSize(collection: Collection<*>) = collection.size

// reified — acceder al tipo en runtime (solo con inline)
inline fun <reified T> List<*>.filterType(): List<T> =
    filterIsInstance<T>()

inline fun <reified T : Any> createInstance(): T =
    T::class.java.getDeclaredConstructor().newInstance()

inline fun <reified T> Iterable<*>.firstOfType(): T? =
    firstOrNull { it is T } as? T

// Uso de reified
val strings: List<String> = listOf(1, "a", 2, "b").filterType()  // ["a", "b"]

// Función genérica con múltiples type params
fun <K, V> Map<K, V>.swap(): Map<V, K> = entries.associate { (k, v) -> v to k }
fun <A, B, C> ((A) -> B).andThen(fn: (B) -> C): (A) -> C = { fn(this(it)) }

// Varianza en el sitio de uso (use-site variance)
fun copyTo(src: List<out Number>, dest: MutableList<in Number>) {
    dest.addAll(src)
}
```

---

## 🏷️ Type aliases

```kotlin
// Alias para tipos complejos
typealias JsonMap = Map<String, Any?>
typealias StringList = List<String>
typealias Predicate<T> = (T) -> Boolean
typealias Transformer<A, B> = (A) -> B
typealias Handler<T> = suspend (T) -> Unit
typealias ResultCallback<T> = (Result<T>) -> Unit
typealias ErrorHandler = (Throwable) -> Unit

// Alias para función con múltiples parámetros
typealias BiConsumer<A, B> = (A, B) -> Unit
typealias TriFunction<A, B, C, R> = (A, B, C) -> R

// Alias para tipos genéricos comunes
typealias UserMap = HashMap<String, User>
typealias NetworkResult<T> = Result<T>

// Uso
val transform: Transformer<String, Int> = { it.length }
val isPositive: Predicate<Int> = { it > 0 }
val data: JsonMap = mapOf("name" to "Juan", "age" to 28)
```
