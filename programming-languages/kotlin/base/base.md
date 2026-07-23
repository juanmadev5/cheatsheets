# Kotlin — Base

---

## 📌 Variables y constantes

```kotlin
// val — inmutable (equivalente a final en Java)
val name: String = "Juan"
val age = 28                     // Inferencia de tipo
val pi = 3.14159

// var — mutable
var count = 0
var message: String = "Hola"
message = "Mundo"                // Reasignación válida

// const — constante en tiempo de compilación (solo primitivos y String, solo top-level o companion)
const val MAX_RETRIES = 3
const val BASE_URL = "https://api.example.com"
const val TIMEOUT_MS = 5_000L   // Separador de miles para legibilidad

// lateinit — inicialización diferida para tipos no-null (solo var, no primitivos)
lateinit var service: MyService
// Verificar si fue inicializado
if (::service.isInitialized) service.doWork()

// Lazy — inicialización diferida e inmutable (hilo-seguro por defecto)
val heavyObject: ExpensiveObject by lazy {
    ExpensiveObject()  // Se evalúa solo la primera vez que se accede
}
val cachedValue: String by lazy(LazyThreadSafetyMode.NONE) {
    // NONE = sin sincronización, más rápido si no hay concurrencia
    computeValue()
}

// Múltiple asignación desde destructuring
val (x, y) = Pair(10, 20)
val (first, second, third) = Triple("a", "b", "c")
val (id, name2, email) = user    // data class

// Declaración de tipo explícita vs inferida
val list: List<String> = listOf("a", "b")
val list2 = listOf("a", "b")     // String inferido de los elementos
```

---

## 📊 Tipos de datos

### Números

```kotlin
// Enteros
val byte: Byte = 127
val short: Short = 32_767
val int: Int = 2_147_483_647
val long: Long = 9_223_372_036_854_775_807L

// Flotantes
val float: Float = 3.14f
val double: Double = 3.141592653589793

// Literales especiales
val hex = 0xFF                   // 255
val binary = 0b1010_1010         // 170
val big = 1_000_000

// Unsigned (Kotlin 1.5+)
val uByte: UByte = 255u
val uInt: UInt = 4_294_967_295u
val uLong: ULong = 18_446_744_073_709_551_615uL

// Conversiones explícitas — Kotlin NO convierte automáticamente
val i: Int = 42
val l: Long = i.toLong()
val d: Double = i.toDouble()
val s: String = i.toString()
val back: Int = "42".toInt()
val safe: Int? = "abc".toIntOrNull()   // null si falla

// Operaciones numéricas
42 / 5          // 8 (división entera)
42 / 5.0        // 8.4 (doble)
42 % 5          // 2 (módulo)
42.0.toInt()    // 42 (trunca)
Math.round(3.7) // 4L
3.7.toInt()     // 3 (trunca, no redondea)

// Infinity y NaN
Double.POSITIVE_INFINITY
Double.NEGATIVE_INFINITY
Double.NaN
Double.isNaN()
Double.isInfinite()
```

---

### Strings, Boolean, Any, Unit, Nothing

```kotlin
// String
val s = "Hola mundo"
val raw = """
    Línea 1
    Línea 2
    Línea 3
""".trimIndent()

val template = "Hola $name, tenés ${age + 1} años"

// Boolean
val t = true
val f = false
val and = t && f
val or = t || f
val not = !t

// Any — supertipo de todos los tipos no-null
val anything: Any = 42
val anything2: Any = "texto"
val anything3: Any = listOf(1, 2, 3)

// Unit — equivalente a void (un solo valor: Unit)
fun greet(): Unit = println("Hola")   // Unit es opcional
fun greet2() = println("Hola")        // Equivalente

// Nothing — tipo sin instancias, función que nunca retorna
fun fail(message: String): Nothing = throw IllegalStateException(message)
fun loop(): Nothing { while (true) {} }

// Uso de Nothing para indicar código inalcanzable
val value = when (status) {
    "ok" -> "Todo bien"
    "error" -> "Algo falló"
    else -> fail("Estado desconocido: $status")  // Nothing — el compilador sabe que no retorna
}
```

---

## 🛡️ Null Safety

```kotlin
// Tipos nullable con ?
val name: String? = null
val length: Int? = name?.length    // null si name es null

// Operador Elvis ?: — valor por defecto si null
val displayName = name ?: "Anónimo"
val len = name?.length ?: 0

// !! — aserción non-null (lanza NullPointerException si es null — EVITAR)
val forced = name!!.length   // Solo usar cuando es imposible que sea null

// Safe cast as?
val num: Any = "no soy un número"
val intValue: Int? = num as? Int    // null si falla el cast (no lanza)

// Let — ejecutar bloque solo si no es null
name?.let { n ->
    println("Nombre: $n")
    println("Longitud: ${n.length}")
}

// Encadenamiento null-safe
val city = user?.address?.city?.uppercase()

// Elvis con throw
val user = userRepository.findById(id)
    ?: throw ResourceNotFoundException("User", id)

// Colecciones con null
val list: List<String?> = listOf("a", null, "b")
val nonNull: List<String> = list.filterNotNull()

// Smart cast — el compilador infiere el tipo después de verificar
val obj: Any = "Hola"
if (obj is String) {
    println(obj.length)  // obj es String aquí — no necesita cast
}

// Smart cast con && 
if (obj is String && obj.length > 3) {
    println(obj.uppercase())  // obj es String aquí
}

// Smart cast con when
when (obj) {
    is String  -> println("String de longitud ${obj.length}")
    is Int     -> println("Int: ${obj * 2}")
    is List<*> -> println("Lista con ${obj.size} elementos")
    null       -> println("Es null")
    else       -> println("Otro tipo: ${obj::class.simpleName}")
}

// requireNotNull / checkNotNull
val nonNullName = requireNotNull(name) { "El nombre no puede ser null" }
val checked = checkNotNull(config) { "Config no inicializada" }

// Funciones null-aware
listOfNotNull("a", null, "b", null, "c")  // ["a", "b", "c"]
```

---

## ➕ Operadores

```kotlin
// Aritméticos
a + b; a - b; a * b; a / b; a % b

// Asignación compuesta
a += b; a -= b; a *= b; a /= b; a %= b

// Comparación
a == b      // Igualdad estructural (llama a equals())
a === b     // Igualdad referencial (mismo objeto en memoria)
a != b
a < b; a > b; a <= b; a >= b

// Rangos
val range = 1..10           // IntRange 1 a 10 (inclusivo)
val exclusive = 1 until 10  // 1 a 9 (10 excluido)
val reversed = 10 downTo 1  // 10, 9, 8... 1
val stepped = 1..20 step 2  // 1, 3, 5... 19
val charRange = 'a'..'z'

// in — verificar membresía
5 in 1..10             // true
'b' in 'a'..'z'        // true
"hola" in listOf("hola", "mundo")  // true
5 !in 1..3             // true

// Destructuring
val (a2, b2) = Pair(1, 2)
val (x, y, z) = Triple(1, 2, 3)
val (id, name3) = user   // data class con componentN()

// Destructuring en for
for ((key, value) in map) {
    println("$key = $value")
}
for ((index, value) in list.withIndex()) {
    println("[$index] $value")
}

// Spread — pasar array como varargs
val args = arrayOf("a", "b", "c")
printAll(*args)   // * expande el array

// Elvis con lanzar excepción
val result = value ?: error("Valor no puede ser null")
val result2 = value ?: throw IllegalStateException("Error")

// Operador invoke — llamar a un objeto como función
class Greeter {
    operator fun invoke(name: String) = println("Hola $name")
}
val greet = Greeter()
greet("Juan")    // Llama a invoke()

// compareTo — usado por <, >, <=, >=
"abc".compareTo("abd")  // -1 (menor), 0 (igual), 1 (mayor)
```

---

## 🔀 Control de flujo

### if como expresión

```kotlin
// if retorna un valor
val max = if (a > b) a else b
val label = if (score >= 90) "Excelente" else if (score >= 70) "Bueno" else "Mejorable"

// if con bloque — el último valor es el retorno
val result = if (condition) {
    doSomething()
    computeValue()   // Este valor es el retorno del bloque
} else {
    otherValue()
}
```

---

### when como expresión

```kotlin
// when — exhaustivo cuando se usa como expresión (retorna valor)
val description = when (status) {
    "active"   -> "Activo"
    "inactive" -> "Inactivo"
    "pending"  -> "Pendiente"
    else       -> "Estado desconocido: $status"
}

// when con múltiples valores por caso
when (day) {
    "Saturday", "Sunday" -> println("Fin de semana")
    "Monday", "Friday"   -> println("Casi fin de semana")
    else                 -> println("Día laboral")
}

// when sin argumento — funciona como if-else encadenado
when {
    score >= 90 -> println("Excelente")
    score >= 70 -> println("Bueno")
    score >= 50 -> println("Regular")
    else        -> println("Insuficiente")
}

// when con tipo
fun describe(obj: Any): String = when (obj) {
    is Int    -> "Entero: $obj"
    is String -> "String de longitud ${obj.length}"
    is Boolean -> if (obj) "verdadero" else "falso"
    is List<*> -> "Lista con ${obj.size} elementos"
    null      -> "Es null"
    else      -> "Tipo: ${obj::class.simpleName}"
}

// when con rangos
when (score) {
    in 90..100 -> "A"
    in 80 until 90 -> "B"
    in 70 until 80 -> "C"
    else -> "F"
}

// when con subject variable (Kotlin 1.7+)
when (val response = fetchData()) {
    is Success -> process(response.data)
    is Error   -> handleError(response.message)
}
```

---

### Bucles

```kotlin
// for — sobre cualquier Iterable
for (i in 1..5) print(i)
for (i in 1 until 5) print(i)       // 1,2,3,4
for (i in 5 downTo 1) print(i)      // 5,4,3,2,1
for (i in 1..10 step 2) print(i)    // 1,3,5,7,9
for (item in list) println(item)
for ((index, value) in list.withIndex()) println("$index: $value")
for ((key, value) in map) println("$key: $value")

// repeat — iterar N veces
repeat(5) { index -> println("Iteración $index") }
repeat(3) { println("Hola") }

// while
var i = 0
while (i < 10) { print(i); i++ }

// do-while
do { processNext() } while (hasMore())

// break y continue con labels
outer@ for (i in 1..5) {
    for (j in 1..5) {
        if (j == 3) continue@outer    // Saltar al siguiente i
        if (i == 4) break@outer       // Salir del bucle externo
        print("$i,$j ")
    }
}

// forEach con return (en lambda el return es de la lambda, no de la función)
list.forEach { item ->
    if (item == "skip") return@forEach  // continue en forEach
    println(item)
}

// forEachIndexed
list.forEachIndexed { index, item ->
    println("[$index] $item")
}
```
