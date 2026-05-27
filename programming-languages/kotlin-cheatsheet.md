# Kotlin — Cheatsheet

---

## 📑 Índice

### Base
- [Variables y constantes — val, var, const, lateinit](#-variables-y-constantes)
- [Tipos de datos — números, String, Boolean, Any, Unit, Nothing](#-tipos-de-datos)
- [Null safety — ?, ?., !!, ?:, let, también smart casts](#️-null-safety)
- [Operadores — aritméticos, in, rangos, spread, destructuring](#-operadores)
- [Control de flujo — if/when/for/while como expresiones](#-control-de-flujo)

### Kotlin moderno — 2.0+
- [when con guards y subject variables](#-when-avanzado--kotlin-20)
- [Value classes — @JvmInline](#-value-classes)
- [Data classes — copy, componentN, desestructuración](#-data-classes)
- [Sealed classes y sealed interfaces](#-sealed-classes-y-sealed-interfaces)
- [Enum classes avanzados](#-enum-classes)

### POO
- [Clases — open, abstract, inner, nested, object](#️-clases)
- [Herencia e interfaces con implementación default](#-herencia-e-interfaces)
- [Companion objects, object expressions, object declarations](#-companion-objects-y-objects)
- [Delegación — by, Lazy, Observable, Vetoable, Map](#-delegación)
- [Extension functions y properties](#-extension-functions-y-properties)
- [Operator overloading](#️-operator-overloading)

### Funcional
- [Lambdas y function types](#-lambdas-y-function-types)
- [inline, noinline, crossinline](#-inline-noinline-crossinline)
- [Scope functions — let, run, with, apply, also](#-scope-functions)
- [Funciones de orden superior — map, filter, fold, reduce](#-funciones-de-orden-superior)
- [Generics — variance, reified, star projection](#-generics)
- [Type aliases e intersection types](#-type-aliases)

### Colecciones
- [List, MutableList — operaciones completas](#-list)
- [Map, MutableMap — operaciones completas](#️-map)
- [Set, MutableSet — operaciones de conjuntos](#-set)
- [Sequence — lazy evaluation](#-sequence--evaluación-lazy)
- [buildList, buildMap, buildSet](#️-builders-de-colecciones)
- [Destructuring y Pair, Triple](#-destructuring-pair-triple)

### Coroutines
- [launch, async, runBlocking, coroutineScope](#-coroutines--base)
- [Dispatchers — IO, Main, Default, Unconfined](#-dispatchers)
- [withContext, withTimeout, withTimeoutOrNull](#-withcontext-y-timeout)
- [Job, Deferred, cancelación y cleanup](#-job-deferred-y-cancelación)
- [Flow — builders, operadores, cold vs hot](#-flow)
- [StateFlow y SharedFlow](#-stateflow-y-sharedflow)
- [Channel — tipos, produce, fan-out/fan-in](#-channel)
- [Exception handling — CoroutineExceptionHandler, SupervisorJob](#️-exception-handling-en-coroutines)
- [Structured concurrency](#-structured-concurrency)

### Standard Library
- [String — métodos, templates, raw strings, buildString](#-string)
- [Regex](#-regex)
- [Result<T> — runCatching, map, recover, getOrElse](#-resultt)
- [Números y conversiones](#-números-y-conversiones)
- [Comparación — Comparable, Comparator, sortedWith](#-comparación)
- [kotlin.io y kotlin.system](#-kotlinio-y-kotlinsystem)

### Interoperabilidad con Java
- [@JvmStatic, @JvmOverloads, @JvmField, @JvmName](#-interoperabilidad-con-java)
- [SAM conversions y platform types](#-sam-conversions-y-platform-types)

### Build
- [build.gradle.kts — completo](#️-buildgradlekts)

### Referencia
- [Idioms y buenas prácticas](#-idioms-y-buenas-prácticas)
- [Patrones de diseño en Kotlin](#-patrones-de-diseño-en-kotlin)

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

---

## 🔥 when avanzado — Kotlin 2.0+

```kotlin
// Guards en when (Kotlin 2.0+) — condición adicional después del pattern
fun process(event: Event): String = when (event) {
    is UserEvent if event.userId > 0 && event.isActive -> "Usuario activo: ${event.userId}"
    is UserEvent if event.userId == 0                  -> "Usuario anónimo"
    is UserEvent                                        -> "Usuario inactivo"
    is SystemEvent if event.severity == "CRITICAL"     -> "🚨 ${event.message}"
    is SystemEvent                                      -> "Sistema: ${event.message}"
    else                                                -> "Evento desconocido"
}

// when exhaustivo con sealed — el compilador verifica todos los casos
sealed interface Result<out T>
data class Success<T>(val data: T) : Result<T>
data class Failure(val error: String, val cause: Throwable? = null) : Result<Nothing>
data object Loading : Result<Nothing>

fun <T> handleResult(result: Result<T>): String = when (result) {
    is Success  -> "Éxito: ${result.data}"
    is Failure  -> "Error: ${result.error}"
    is Loading  -> "Cargando..."
    // No necesita else — sealed garantiza exhaustividad
}

// when con captura de variable y guard
fun categorize(obj: Any?): String = when {
    obj == null            -> "null"
    obj is String && obj.isBlank() -> "string vacío"
    obj is String && obj.length > 100 -> "string largo"
    obj is String          -> "string: $obj"
    obj is Int && obj < 0  -> "negativo: $obj"
    obj is Int             -> "positivo: $obj"
    obj is List<*>         -> "lista[${obj.size}]"
    else                   -> "otro: ${obj::class.simpleName}"
}
```

---

## 💎 Value classes

```kotlin
// @JvmInline value class — wrapper sin overhead en runtime
// El compilador reemplaza el wrapper con el tipo subyacente en bytecode
@JvmInline
value class UserId(val value: Long) {
    init {
        require(value > 0) { "UserId debe ser positivo" }
    }

    fun isValid(): Boolean = value > 0
    override fun toString() = "UserId($value)"
}

@JvmInline
value class Email(val value: String) {
    init {
        require(value.contains("@")) { "Email inválido: $value" }
    }

    val domain: String get() = value.substringAfter("@")
    val local: String get() = value.substringBefore("@")
}

@JvmInline
value class Password(private val hash: String) {
    fun matches(plain: String): Boolean = verifyHash(plain, hash)
}

// Uso — type safety sin overhead de objeto extra
val userId = UserId(42L)
val email = Email("juan@example.com")

fun findUser(id: UserId, email: Email) { ... }   // No se pueden confundir tipos
findUser(userId, email)                           // ✅
// findUser(email, userId)                        // ❌ Error de compilación

// Value class con interfaz
interface Printable { fun print() }

@JvmInline
value class Name(val value: String) : Printable {
    override fun print() = println("Name: $value")
}

// Múltiples propiedades — value class SOLO puede tener UNA propiedad
// Para múltiples, usar data class
```

---

## 📦 Data classes

```kotlin
// data class — genera equals, hashCode, toString, copy, componentN
data class Product(
    val id: Long,
    val name: String,
    val price: Double,
    val category: String,
    val active: Boolean = true,
    val tags: List<String> = emptyList()
)

val p1 = Product(1L, "Laptop", 999.99, "Electronics")
val p2 = p1.copy(price = 899.99)          // Copia con precio modificado
val p3 = p1.copy(name = "Tablet", price = 499.99)

// Igualdad estructural — compara por contenido
p1 == p2    // false (precios diferentes)
p1 === p2   // false (objetos distintos)

// toString automático
println(p1)  // Product(id=1, name=Laptop, price=999.99, category=Electronics, active=true, tags=[])

// Destructuring — componentN() generado automáticamente
val (id, name, price) = p1
val (_, _, price2, category) = p1   // _ ignora componentes

// Desestructuración en for
val products = listOf(p1, p2)
for ((id2, name2) in products) {
    println("$id2: $name2")
}

// data class con validación en init
data class User(
    val id: Long,
    val name: String,
    val email: String,
    val age: Int
) {
    init {
        require(name.isNotBlank()) { "El nombre no puede estar vacío" }
        require(email.contains("@")) { "Email inválido: $email" }
        require(age in 0..150) { "Edad inválida: $age" }
    }

    val displayName: String get() = "$name <$email>"
}

// data class anidada
data class Address(
    val street: String,
    val city: String,
    val country: String
)

data class Person(
    val id: Long,
    val name: String,
    val address: Address
)

// Actualizar objeto anidado inmutable
val updated = person.copy(
    address = person.address.copy(city = "Buenos Aires")
)

// data object — singleton con equals/hashCode/toString (Kotlin 1.9+)
data object DatabaseConnection {
    val url = "jdbc:postgresql://localhost/db"
}

// Excluir campo de equals/hashCode
data class Entity(
    val id: Long,
    val name: String,
    @Transient val cachedValue: String = ""  // excluido de equals
) {
    // Para excluir campo: moverlo al body del constructor
    // o usar @Transient (depende del framework)
}
```

---

## 🔒 Sealed classes y sealed interfaces

```kotlin
// sealed class — jerarquía cerrada en el mismo paquete
sealed class NetworkResult<out T> {
    data class Success<T>(val data: T) : NetworkResult<T>()
    data class Error(val code: Int, val message: String) : NetworkResult<Nothing>()
    data object Loading : NetworkResult<Nothing>()
    data class Exception(val throwable: Throwable) : NetworkResult<Nothing>()
}

// sealed interface — más flexible (permite implementar múltiples)
sealed interface UiState
data object Idle : UiState
data object Loading : UiState
data class Content<T>(val data: T) : UiState
data class Error(val message: String, val retryable: Boolean = true) : UiState

// when exhaustivo — el compilador verifica que se cubran todos los casos
fun handleState(state: UiState): String = when (state) {
    is Idle    -> "En espera"
    is Loading -> "Cargando..."
    is Content<*> -> "Contenido: ${state.data}"
    is Error   -> if (state.retryable) "Error (reintentar)" else "Error fatal"
}

// Sealed con herencia profunda
sealed class AppError(val message: String) {
    sealed class NetworkError(message: String) : AppError(message) {
        class Timeout(val durationMs: Long) : NetworkError("Timeout después de ${durationMs}ms")
        class NoConnection : NetworkError("Sin conexión a internet")
        class ServerError(val code: Int) : NetworkError("Error del servidor: $code")
    }
    sealed class AuthError(message: String) : AppError(message) {
        class Unauthorized : AuthError("No autenticado")
        class Forbidden : AuthError("Sin permisos")
        class TokenExpired : AuthError("Token expirado")
    }
    class UnknownError(cause: Throwable) : AppError(cause.message ?: "Error desconocido")
}

// Manejo exhaustivo con when y guard (Kotlin 2.0)
fun handleError(error: AppError): String = when (error) {
    is AppError.NetworkError.Timeout if error.durationMs > 30_000 -> "Timeout muy largo"
    is AppError.NetworkError.Timeout -> "Timeout"
    is AppError.NetworkError.NoConnection -> "Sin internet"
    is AppError.NetworkError.ServerError if error.code == 503 -> "Servicio no disponible"
    is AppError.NetworkError.ServerError -> "Error ${error.code}"
    is AppError.AuthError.TokenExpired -> "Sesión expirada — volvé a iniciar sesión"
    is AppError.AuthError -> error.message
    is AppError.UnknownError -> "Error inesperado"
}

// Extension functions sobre sealed
fun NetworkResult<*>.isSuccess() = this is NetworkResult.Success
fun NetworkResult<*>.isLoading() = this is NetworkResult.Loading

fun <T, R> NetworkResult<T>.map(transform: (T) -> R): NetworkResult<R> = when (this) {
    is NetworkResult.Success   -> NetworkResult.Success(transform(data))
    is NetworkResult.Error     -> this
    is NetworkResult.Loading   -> this
    is NetworkResult.Exception -> this
}

fun <T> NetworkResult<T>.getOrNull(): T? =
    if (this is NetworkResult.Success) data else null

fun <T> NetworkResult<T>.getOrDefault(default: T): T =
    if (this is NetworkResult.Success) data else default

fun <T> NetworkResult<T>.getOrThrow(): T = when (this) {
    is NetworkResult.Success   -> data
    is NetworkResult.Error     -> throw RuntimeException("[$code] $message")
    is NetworkResult.Exception -> throw throwable
    else                       -> throw IllegalStateException("No hay datos")
}
```

---

## 🎭 Enum classes

```kotlin
// Enum básico
enum class Direction { NORTH, SOUTH, EAST, WEST }

// Enum con propiedades y métodos
enum class HttpStatus(val code: Int, val description: String) {
    OK(200, "Success"),
    CREATED(201, "Created"),
    BAD_REQUEST(400, "Bad Request"),
    UNAUTHORIZED(401, "Unauthorized"),
    FORBIDDEN(403, "Forbidden"),
    NOT_FOUND(404, "Not Found"),
    INTERNAL_ERROR(500, "Internal Server Error");

    val isSuccess: Boolean get() = code in 200..299
    val isError: Boolean get() = code >= 400
    val isClientError: Boolean get() = code in 400..499
    val isServerError: Boolean get() = code >= 500

    companion object {
        fun fromCode(code: Int): HttpStatus? = entries.find { it.code == code }
        fun fromCodeOrDefault(code: Int, default: HttpStatus = INTERNAL_ERROR) =
            fromCode(code) ?: default
    }
}

// Enum con comportamiento abstracto — cada valor implementa diferente
enum class Operation(val symbol: String) {
    ADD("+") {
        override fun apply(a: Double, b: Double) = a + b
    },
    SUBTRACT("-") {
        override fun apply(a: Double, b: Double) = a - b
    },
    MULTIPLY("*") {
        override fun apply(a: Double, b: Double) = a * b
    },
    DIVIDE("/") {
        override fun apply(a: Double, b: Double) {
            require(b != 0.0) { "No se puede dividir por cero" }
            return a / b
        }
    };

    abstract fun apply(a: Double, b: Double): Double
}

// Propiedades built-in de enums
Direction.NORTH.name       // "NORTH"
Direction.NORTH.ordinal    // 0
Direction.entries          // [NORTH, SOUTH, EAST, WEST] (Kotlin 1.9+, reemplaza values())
Direction.valueOf("SOUTH") // Direction.SOUTH (lanza si no existe)

// when exhaustivo con enum (sin else)
fun describe(dir: Direction): String = when (dir) {
    Direction.NORTH -> "Norte"
    Direction.SOUTH -> "Sur"
    Direction.EAST  -> "Este"
    Direction.WEST  -> "Oeste"
}

// Enum con interfaz
interface Localizable { fun toLocalString(): String }

enum class Season : Localizable {
    SPRING {
        override fun toLocalString() = "Primavera"
    },
    SUMMER {
        override fun toLocalString() = "Verano"
    },
    AUTUMN {
        override fun toLocalString() = "Otoño"
    },
    WINTER {
        override fun toLocalString() = "Invierno"
    }
}
```

---

## 🏛️ Clases

```kotlin
// Clase básica con constructor primario
class Person(
    val name: String,           // val → getter generado
    var age: Int,               // var → getter + setter generados
    private val email: String   // private
) {
    // Bloque init — ejecutado al construir
    init {
        require(name.isNotBlank()) { "Nombre requerido" }
        require(age >= 0) { "Edad inválida: $age" }
    }

    // Constructor secundario — debe delegar al primario
    constructor(name: String, email: String) : this(name, 0, email)

    // Propiedades calculadas
    val displayName: String get() = "$name ($email)"
    val isAdult: Boolean get() = age >= 18

    // Propiedad con setter personalizado
    var score: Int = 0
        get() = field
        set(value) {
            require(value >= 0) { "Score no puede ser negativo" }
            field = value
        }

    // Métodos
    fun greet() = "Hola, soy $name"
    override fun toString() = "Person(name=$name, age=$age)"
}

// open — permite herencia (las clases son final por defecto en Kotlin)
open class Animal(val name: String) {
    open fun makeSound() = println("...")
    open val type: String get() = "Animal"

    // fun sin open — no se puede sobreescribir
    fun breathe() = println("$name respira")
}

// abstract — no se puede instanciar directamente
abstract class Shape {
    abstract fun area(): Double
    abstract fun perimeter(): Double

    // Implementación por defecto
    fun describe() = "Figura con área ${area()} y perímetro ${perimeter()}"
}

// inner class — tiene referencia a la clase externa
class Outer(val value: Int) {
    inner class Inner {
        fun getOuterValue() = value   // Accede a Outer.value
    }
}

// nested class — sin referencia a la clase externa (equivalente a static en Java)
class Container {
    class Item(val id: Int)   // No puede acceder a Container
    val item = Item(1)
}

// object — singleton (una sola instancia)
object AppConfig {
    const val VERSION = "1.0.0"
    var debug = false

    fun getBaseUrl() = if (debug) "https://dev.api.com" else "https://api.com"
}
AppConfig.debug = true
AppConfig.getBaseUrl()

// Visibilidad (de más a menos restrictivo)
// private   → solo en el archivo/clase
// internal  → en el mismo módulo
// protected → en la clase y subclases
// public    → (default) en todos lados

class MyClass {
    private val privateField = 1
    internal val internalField = 2
    protected val protectedField = 3
    val publicField = 4      // public por defecto
}
```

---

## 🧬 Herencia e interfaces

```kotlin
// Herencia — : ExtendedClass(args)
class Dog(name: String, val breed: String) : Animal(name) {
    override fun makeSound() = println("$name: ¡Guau!")
    override val type: String get() = "Perro"

    fun fetch() = println("$name trae la pelota")
}

// Llamar al método del padre
class GuideDog(name: String) : Dog(name, "Labrador") {
    override fun makeSound() {
        super.makeSound()           // Llama al Dog.makeSound()
        println("(guía silenciosa)")
    }
}

// Interfaz — puede tener implementaciones por defecto
interface Flyable {
    val maxAltitude: Int            // Abstracta — debe implementarse
    fun fly() = println("Volando") // Default — puede sobreescribirse

    // Propiedad con implementación
    val altitudeDescription: String
        get() = "Altitud máxima: $maxAltitude metros"
}

interface Swimmable {
    fun swim() = println("Nadando")
    fun dive(depth: Int) = println("Buceando a ${depth}m")
}

// Implementar múltiples interfaces
class Duck : Animal("Pato"), Flyable, Swimmable {
    override val maxAltitude = 100

    override fun makeSound() = println("Pato: ¡Cuac!")

    // Sobreescribir implementación default
    override fun fly() = println("Pato volando bajo")

    // Swimmable se usa con implementación por defecto
}

// Resolver conflicto de implementaciones
interface A { fun method() = println("A") }
interface B { fun method() = println("B") }

class C : A, B {
    override fun method() {
        super<A>.method()   // Llamar explícitamente a A
        super<B>.method()   // Llamar explícitamente a B
    }
}

// by — delegar implementación de interfaz a otro objeto
class RobotDog(private val delegate: Dog) : Animal(delegate.name), Flyable by FlyingModule() {
    override fun makeSound() = delegate.makeSound()   // Delegar explícitamente
}
```

---

## 🤝 Companion objects y objects

```kotlin
// companion object — equivalente a static en Java
class Product(val id: Long, val name: String, val price: Double) {

    companion object {
        // "Factory methods" — patrón idiomático en Kotlin
        fun create(name: String, price: Double): Product =
            Product(System.currentTimeMillis(), name, price)

        fun fromJson(json: String): Product {
            // parsear JSON...
            return Product(1L, "parsed", 0.0)
        }

        // Constantes
        const val MAX_PRICE = 999_999.99
        val DEFAULT = Product(0L, "Default", 0.0)

        // Implementar interfaz en companion
        fun empty() = Product(0L, "", 0.0)
    }
}

// Acceso
val p = Product.create("Laptop", 999.99)
val maxPrice = Product.MAX_PRICE
val empty = Product.empty()

// companion object con nombre (raro, pero posible)
class Factory {
    companion object Builder {
        fun build() = Factory()
    }
}
Factory.build()
Factory.Builder.build()  // Ambas formas funcionan

// object expression — anonymous object (equivalente a anonymous class en Java)
val comparator = object : Comparator<String> {
    override fun compare(a: String, b: String) = a.length - b.length
}

// Más idiomático con lambda
val comparator2 = Comparator<String> { a, b -> a.length - b.length }

// object expression puede extender clase e implementar interfaces
val handler = object : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) = println("Click")
    override fun mouseEntered(e: MouseEvent) = println("Enter")
}

// object declaration — singleton nombrado
object Logger {
    private val logs = mutableListOf<String>()

    fun log(message: String) {
        logs.add("[${System.currentTimeMillis()}] $message")
        println(message)
    }

    fun getLogs(): List<String> = logs.toList()
    fun clear() = logs.clear()
}

Logger.log("App iniciada")
```

---

## 🔗 Delegación

```kotlin
// by — delegar implementación de interfaz
interface Storage {
    fun save(key: String, value: String)
    fun load(key: String): String?
}

class InMemoryStorage : Storage {
    private val map = mutableMapOf<String, String>()
    override fun save(key: String, value: String) { map[key] = value }
    override fun load(key: String) = map[key]
}

// EncryptedStorage delega a InMemoryStorage y solo modifica lo necesario
class EncryptedStorage(private val base: InMemoryStorage) : Storage by base {
    override fun save(key: String, value: String) {
        base.save(key, encrypt(value))  // Sobreescribir solo save
    }
    private fun encrypt(value: String) = value.reversed()  // Simplificado
}

// Lazy — inicialización diferida y hilo-segura
val heavyResource: HeavyResource by lazy {
    HeavyResource()  // Se crea la primera vez que se accede
}

// Observable — notificar cambios
import kotlin.properties.Delegates

var observedValue: String by Delegates.observable("inicial") { property, old, new ->
    println("${property.name}: $old → $new")
}
observedValue = "nuevo"  // Imprime: observedValue: inicial → nuevo

// Vetoable — validar antes de cambiar
var positiveNumber: Int by Delegates.vetoable(0) { _, _, new ->
    new >= 0  // Retornar true para aceptar, false para rechazar
}
positiveNumber = 5    // Aceptado: 5
positiveNumber = -1   // Rechazado: sigue siendo 5

// notNull — variable non-null sin valor inicial (alternativa a lateinit para primitivos)
var id: Long by Delegates.notNull()
// id = 42  // Debe asignarse antes de leer

// Map como backing store para propiedades
class Config(private val map: Map<String, Any?>) {
    val host: String by map          // Lee "host" del mapa
    val port: Int by map
    val debug: Boolean by map
}

val config = Config(mapOf("host" to "localhost", "port" to 8080, "debug" to true))
println(config.host)   // "localhost"
println(config.port)   // 8080

// Delegado personalizado
import kotlin.reflect.KProperty

class Trimmed {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String = _value.trim()
    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        _value = value
    }
    private var _value = ""
}

class Form {
    var username: String by Trimmed()
    var email: String by Trimmed()
}

val form = Form()
form.username = "  Juan  "
println(form.username)  // "Juan" (trimmed)
```

---

## 🔌 Extension functions y properties

```kotlin
// Extension function — agregar funciones a tipos existentes sin herencia
fun String.capitalize(): String =
    if (isEmpty()) this else "${this[0].uppercaseChar()}${substring(1)}"

fun String.isValidEmail(): Boolean =
    Regex("^[\\w.+-]+@[\\w-]+\\.[\\w.]+$").matches(this)

fun String.truncate(maxLength: Int, ellipsis: String = "..."): String =
    if (length <= maxLength) this else "${substring(0, maxLength)}$ellipsis"

fun String.toSnakeCase(): String = replace(Regex("([A-Z])"), "_$1").lowercase().trimStart('_')

fun String.countWords(): Int = trim().split(Regex("\\s+")).size

// Extension sobre tipos nullable
fun String?.isNullOrBlankTrimmed() = this == null || this.trim().isBlank()

fun String?.orDefault(default: String) = if (isNullOrEmpty()) default else this!!

// Extension sobre colecciones
fun <T> List<T>.second(): T {
    if (size < 2) throw NoSuchElementException("Lista tiene menos de 2 elementos")
    return this[1]
}

fun <T> List<T>.firstOrDefault(default: T): T = if (isEmpty()) default else first()

fun <T, K> List<T>.groupByCount(keySelector: (T) -> K): Map<K, Int> =
    groupBy(keySelector).mapValues { it.value.size }

fun <T : Comparable<T>> List<T>.median(): T? {
    if (isEmpty()) return null
    val sorted = sorted()
    return sorted[sorted.size / 2]
}

// Extension sobre Int y Long
fun Int.seconds() = this * 1000L
fun Int.minutes() = this * 60_000L
fun Int.hours() = this * 3_600_000L

fun Long.toHumanReadable(): String = when {
    this < 1_000      -> "${this}B"
    this < 1_000_000  -> "${this / 1_000}KB"
    this < 1_000_000_000 -> "${this / 1_000_000}MB"
    else              -> "${this / 1_000_000_000}GB"
}

// Extension property
val String.isPalindrome: Boolean get() = this == this.reversed()
val String.wordCount: Int get() = trim().split(Regex("\\s+")).size
val Int.isEven: Boolean get() = this % 2 == 0
val Int.isPrime: Boolean get() {
    if (this < 2) return false
    return (2..Math.sqrt(this.toDouble()).toInt()).none { this % it == 0 }
}
val <T> List<T>.lastIndex: Int get() = size - 1  // Ya existe en stdlib, ejemplo

// Extension sobre companion object
class MyClass {
    companion object
}
fun MyClass.Companion.create() = MyClass()

// Uso
"hola mundo".capitalize()           // "Hola mundo"
"test@email.com".isValidEmail()     // true
"Texto largo".truncate(5)           // "Texto..."
"helloWorld".toSnakeCase()          // "hello_world"
"aba".isPalindrome                   // true
5.seconds()                          // 5000L
1_500_000L.toHumanReadable()        // "1MB"
```

---

## ⚙️ Operator overloading

```kotlin
data class Vector(val x: Double, val y: Double) {
    // Operadores aritméticos
    operator fun plus(other: Vector) = Vector(x + other.x, y + other.y)
    operator fun minus(other: Vector) = Vector(x - other.x, y - other.y)
    operator fun times(scalar: Double) = Vector(x * scalar, y * scalar)
    operator fun div(scalar: Double) = Vector(x / scalar, y / scalar)
    operator fun unaryMinus() = Vector(-x, -y)   // -vector

    // Comparación
    operator fun compareTo(other: Vector): Int =
        magnitude.compareTo(other.magnitude)

    // Contenido
    operator fun contains(value: Double) = value == x || value == y

    // Indexing
    operator fun get(index: Int): Double = when (index) {
        0 -> x; 1 -> y
        else -> throw IndexOutOfBoundsException("Index: $index")
    }
    operator fun set(index: Int, value: Double) { /* si fuera mutable */ }

    // Invoke — llamar como función
    operator fun invoke(scale: Double) = this * scale

    // Iterator — hacer el objeto iterable
    operator fun iterator() = listOf(x, y).iterator()

    val magnitude: Double get() = Math.sqrt(x * x + y * y)
    override fun toString() = "Vector($x, $y)"
}

val v1 = Vector(1.0, 2.0)
val v2 = Vector(3.0, 4.0)

val sum = v1 + v2           // Vector(4.0, 6.0)
val diff = v1 - v2          // Vector(-2.0, -2.0)
val scaled = v1 * 2.0       // Vector(2.0, 4.0)
val neg = -v1               // Vector(-1.0, -2.0)
val contained = 1.0 in v1  // true
val x = v1[0]               // 1.0
val scaled2 = v1(3.0)       // v1.invoke(3.0)

for (component in v1) println(component)   // 1.0, 2.0

// Operadores de asignación compuesta
data class Counter(var value: Int = 0) {
    operator fun plusAssign(amount: Int) { value += amount }
    operator fun minusAssign(amount: Int) { value -= amount }
    operator fun inc() = Counter(value + 1)   // ++counter
    operator fun dec() = Counter(value - 1)   // --counter
}
```

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

---

## ⚡ Coroutines — base

```kotlin
// Dependencias build.gradle.kts
// implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.9.0")

// runBlocking — bloquea el thread actual hasta que terminen las coroutines
// Solo usar en main() o en tests
fun main() = runBlocking {
    println("Inicio")
    launch { delay(100); println("Coroutine 1") }
    launch { delay(50); println("Coroutine 2") }
    println("Fin")
}
// Salida: Inicio, Fin, Coroutine 2, Coroutine 1

// launch — fire-and-forget, retorna Job
val job = CoroutineScope(Dispatchers.IO).launch {
    doHeavyWork()
}

// async — retorna Deferred<T> con resultado
val deferred = CoroutineScope(Dispatchers.IO).async {
    computeValue()   // Tipo inferido
}
val result = deferred.await()   // Suspende hasta que esté disponible

// coroutineScope — crea scope que espera a todos sus hijos
// Si cualquier hijo falla, cancela los demás
suspend fun doWork() = coroutineScope {
    val r1 = async { fetchUsers() }
    val r2 = async { fetchProducts() }
    Pair(r1.await(), r2.await())   // Paralelo
}

// supervisorScope — si un hijo falla, los demás continúan
suspend fun doSupervisedWork() = supervisorScope {
    val r1 = async { riskyOperation1() }
    val r2 = async { riskyOperation2() }

    try {
        Pair(r1.await(), r2.await())
    } catch (e: Exception) {
        Pair(null, r2.await())   // r2 puede seguir aunque r1 falle
    }
}

// Suspending function — función que puede suspenderse sin bloquear el thread
suspend fun fetchData(): String {
    delay(1000)         // Suspende, no bloquea
    return "datos"
}

// GlobalScope — NO usar en producción (no tiene structured concurrency)
GlobalScope.launch { ... }   // ❌ — evitar
```

---

## 🚀 Dispatchers

```kotlin
// Dispatchers.Main — UI thread (Android/JavaFX)
CoroutineScope(Dispatchers.Main).launch {
    updateUI()   // Solo en plataformas con Main dispatcher
}

// Dispatchers.IO — operaciones de I/O (red, disco)
// Pool de threads optimizado para I/O bloqueante
CoroutineScope(Dispatchers.IO).launch {
    val data = readFile("/datos.txt")
    val response = httpClient.get("https://api.example.com")
}

// Dispatchers.Default — CPU intensivo
// Pool de threads igual al número de CPUs
CoroutineScope(Dispatchers.Default).launch {
    val sorted = largeList.sortedWith(complexComparator)
    val parsed = parseHugeJson(jsonString)
}

// Dispatchers.Unconfined — hereda el thread del llamador (raro, evitar)
// Solo para testing o casos especiales

// newSingleThreadContext — thread dedicado (costoso, cerrar cuando no se use)
val singleThread = newSingleThreadContext("MyThread")
CoroutineScope(singleThread).launch {
    // Siempre en el mismo thread
}
singleThread.close()   // Liberar el thread

// limitedParallelism — limitar concurrencia
val limitedIO = Dispatchers.IO.limitedParallelism(4)

// withContext — cambiar dispatcher dentro de una coroutine
suspend fun processData(): String = withContext(Dispatchers.Default) {
    // Procesamiento CPU intensivo
    val result = heavyComputation()
    withContext(Dispatchers.IO) {
        // Guardar resultado en disco
        saveToFile(result)
    }
    result
}
```

---

## ⏰ withContext y timeout

```kotlin
// withContext — cambiar dispatcher y esperar resultado
suspend fun fetchAndProcess(): String {
    val raw = withContext(Dispatchers.IO) { fetchFromNetwork() }
    return withContext(Dispatchers.Default) { processData(raw) }
}

// withTimeout — cancelar si supera el tiempo límite
suspend fun safeRequest(): String? = withTimeoutOrNull(5000L) {
    // Si tarda más de 5 segundos, retorna null
    httpClient.get("https://api.example.com/data")
}

// withTimeout — lanza TimeoutCancellationException
suspend fun strictRequest(): String {
    return withTimeout(3000L) {
        httpClient.get("https://api.example.com/data")
    }
}

// Reintento con timeout
suspend fun <T> retryWithTimeout(
    times: Int = 3,
    timeout: Long = 5000L,
    delay: Long = 1000L,
    block: suspend () -> T
): T {
    repeat(times - 1) { attempt ->
        try {
            return withTimeout(timeout) { block() }
        } catch (e: Exception) {
            if (e is CancellationException) throw e
            delay(delay * (attempt + 1))   // Backoff exponencial
        }
    }
    return withTimeout(timeout) { block() }
}
```

---

## 🔧 Job, Deferred y cancelación

```kotlin
// Job — handle de una coroutine
val job = scope.launch {
    try {
        repeat(1000) { i ->
            delay(100)
            println("Trabajo $i")
        }
    } finally {
        println("Limpieza al cancelar")   // Siempre se ejecuta
    }
}

job.cancel()                 // Cancelar
job.cancel(CancellationException("Timeout"))  // Con mensaje
job.join()                   // Esperar a que termine
job.cancelAndJoin()          // Cancelar y esperar
job.isActive                 // true si está corriendo
job.isCancelled
job.isCompleted

// Cancelación cooperativa — la coroutine debe verificar
suspend fun heavyWork() {
    for (i in 1..1000) {
        ensureActive()         // Lanza CancellationException si fue cancelada
        // o: yield() — suspende y permite cancelación
        doStep(i)
    }
}

// isActive — verificar manualmente
suspend fun heavyWorkManual() {
    var i = 0
    while (isActive) {        // Verificar en el loop
        doStep(i++)
    }
}

// Deferred<T> — resultado de async
val deferred: Deferred<Int> = scope.async { computeValue() }
val value = deferred.await()      // Suspende hasta que esté listo
deferred.cancel()
deferred.isCompleted

// Esperar múltiples Deferred
val results = awaitAll(
    async { fetchUsers() },
    async { fetchProducts() }
)

// Job padre-hijo — cancelar el padre cancela los hijos
val parentJob = scope.launch {
    val child1 = launch { task1() }
    val child2 = launch { task2() }
    // Si se cancela parentJob, child1 y child2 también se cancelan
}

// SupervisorJob — el fallo de un hijo no afecta a los demás
val supervisor = SupervisorJob()
val scope = CoroutineScope(Dispatchers.IO + supervisor)
scope.launch { failableTask1() }   // Si falla, no afecta a failableTask2
scope.launch { failableTask2() }
```

---

## 🌊 Flow

```kotlin
// Flow — stream de datos frío (cold) — se ejecuta para cada colector
fun numbersFlow(): Flow<Int> = flow {
    for (i in 1..5) {
        delay(100)
        emit(i)          // Emitir valor
    }
}

// Colectar el flow
numbersFlow().collect { value ->
    println("Recibido: $value")
}

// Builders de Flow
flowOf(1, 2, 3, 4, 5)               // Flow de valores fijos
listOf(1, 2, 3).asFlow()            // Iterable → Flow
(1..10).asFlow()                    // Rango → Flow

// Flow con contexto
val ioFlow = flow {
    val data = withContext(Dispatchers.IO) { fetchFromDb() }
    emit(data)
}.flowOn(Dispatchers.IO)           // Cambiar contexto del productor

// Operadores de Flow (lazy, como Sequence)
numbersFlow()
    .map { it * 2 }
    .filter { it > 4 }
    .take(3)
    .onEach { println("Procesando: $it") }
    .catch { e -> emit(-1) }        // Manejar errores
    .onStart { emit(0) }            // Emitir antes del primero
    .onCompletion { println("Flow terminado") }
    .collect { println(it) }

// Terminal operators
flow.first()                        // Primer elemento
flow.firstOrNull()
flow.last()
flow.single()
flow.toList()                       // Colectar en lista
flow.toSet()
flow.count()
flow.fold(0) { acc, v -> acc + v }
flow.reduce { acc, v -> acc + v }

// transform — más flexible que map (puede emitir múltiples valores)
flow.transform { value ->
    emit("Procesando $value")
    if (value > 3) emit("$value es grande")
}

// flatMapLatest — cancela el anterior al llegar uno nuevo
searchQuery.flatMapLatest { query ->
    flow { emit(search(query)) }
}

// flatMapMerge — ejecuta en paralelo (no cancela)
ids.asFlow().flatMapMerge { id ->
    flow { emit(fetchById(id)) }
}

// flatMapConcat — secuencial (espera cada uno)
ids.asFlow().flatMapConcat { id ->
    flow { emit(fetchById(id)) }
}

// combine — combinar dos flows
val combined = combine(flow1, flow2) { a, b -> "$a-$b" }

// zip — emparejar elemento a elemento
val zipped = flow1.zip(flow2) { a, b -> "$a+$b" }

// merge — unir múltiples flows
val merged = merge(flow1, flow2, flow3)

// buffer — almacenar en buffer mientras el consumidor procesa
numbersFlow()
    .buffer(10)             // Buffer de 10 elementos
    .collect { process(it) }

// conflate — solo el último si el consumidor es lento
fastProducer.conflate().collect { slowConsumer(it) }

// debounce — emitir solo si hubo silencio por N ms
searchInput.debounce(300).collect { query -> search(query) }

// distinctUntilChanged — no emitir si el valor no cambió
stateFlow.distinctUntilChanged().collect { updateUI(it) }

// retry / retryWhen
apiFlow.retry(3) { cause ->
    cause is IOException   // Solo reintentar en IOExceptions
}.collect { ... }

apiFlow.retryWhen { cause, attempt ->
    delay(1000 * attempt)
    attempt < 3 && cause is IOException
}.collect { ... }
```

---

## 🔥 StateFlow y SharedFlow

```kotlin
// StateFlow — estado con valor actual siempre disponible
// Hot flow — siempre activo, nuevo suscriptor recibe el último valor
class MyViewModel {
    private val _uiState = MutableStateFlow<UiState>(UiState.Loading)
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()

    fun loadData() {
        scope.launch {
            _uiState.value = UiState.Loading
            try {
                val data = repository.getData()
                _uiState.value = UiState.Success(data)
            } catch (e: Exception) {
                _uiState.value = UiState.Error(e.message ?: "Error")
            }
        }
    }

    // update — actualización atómica
    fun incrementCount() {
        _uiState.update { current ->
            (current as? UiState.Success)?.copy(count = current.count + 1)
                ?: current
        }
    }
}

// Colectar StateFlow
viewModel.uiState.collect { state ->
    when (state) {
        is UiState.Loading -> showLoading()
        is UiState.Success -> showContent(state.data)
        is UiState.Error   -> showError(state.message)
    }
}

// SharedFlow — para eventos de difusión (hot flow sin valor actual)
class EventBus {
    private val _events = MutableSharedFlow<AppEvent>(
        replay = 0,              // Sin historial para nuevos suscriptores
        extraBufferCapacity = 16,
        onBufferOverflow = BufferOverflow.DROP_OLDEST
    )
    val events: SharedFlow<AppEvent> = _events.asSharedFlow()

    suspend fun emit(event: AppEvent) = _events.emit(event)
    fun tryEmit(event: AppEvent) = _events.tryEmit(event)
}

// SharedFlow con replay — nuevos suscriptores reciben los últimos N eventos
val sharedWithHistory = MutableSharedFlow<String>(replay = 3)

// Convertir Flow frío en SharedFlow caliente
val hotFlow = coldFlow.shareIn(
    scope = viewModelScope,
    started = SharingStarted.WhileSubscribed(5000),  // Activo mientras haya suscriptores
    replay = 1
)

// Convertir Flow frío en StateFlow
val stateFlow = coldFlow.stateIn(
    scope = viewModelScope,
    started = SharingStarted.WhileSubscribed(5000),
    initialValue = UiState.Loading
)

// SharingStarted opciones
SharingStarted.Eagerly           // Inicia inmediatamente, nunca para
SharingStarted.Lazily            // Inicia al primer suscriptor, nunca para
SharingStarted.WhileSubscribed() // Inicia/para con suscriptores
SharingStarted.WhileSubscribed(stopTimeoutMillis = 5000)  // 5s de gracia
```

---

## 📡 Channel

```kotlin
// Channel — comunicación entre coroutines (like a queue)
val channel = Channel<Int>(capacity = 10)

// Producer
launch {
    for (i in 1..5) {
        channel.send(i)      // Suspende si el buffer está lleno
    }
    channel.close()          // Señaliza que no habrá más datos
}

// Consumer
launch {
    for (value in channel) { // Itera hasta que el canal se cierra
        println("Recibido: $value")
    }
}
// O:
launch {
    while (!channel.isClosedForReceive) {
        val value = channel.receive()   // Suspende hasta que haya datos
        println(value)
    }
}

// Channel capacities
Channel<Int>()                          // RENDEZVOUS (0) — send suspende hasta receive
Channel<Int>(Channel.BUFFERED)          // Buffer por defecto (64)
Channel<Int>(Channel.UNLIMITED)         // Sin límite
Channel<Int>(Channel.CONFLATED)         // Solo guarda el último
Channel<Int>(10)                        // Buffer fijo

// produce — builder para Channel como producer
fun CoroutineScope.produceNumbers() = produce<Int> {
    for (i in 1..5) send(i)
}

val numbers = produceNumbers()
numbers.consumeEach { println(it) }

// Fan-out — múltiples consumidores del mismo channel
val producer = produce { repeat(5) { send(it); delay(100) } }
repeat(3) { workerId ->
    launch {
        for (msg in producer) {
            println("Worker $workerId: $msg")
        }
    }
}

// Fan-in — múltiples productores en el mismo channel
val output = Channel<String>()
launch { for (i in 1..3) output.send("A$i") }
launch { for (i in 1..3) output.send("B$i") }
launch {
    repeat(6) { println(output.receive()) }
    output.close()
}

// ticker — emite unidades cada N milisegundos
val ticker = ticker(delayMillis = 1000, initialDelayMillis = 0)
repeat(5) {
    ticker.receive()
    println("tick ${System.currentTimeMillis()}")
}
ticker.cancel()
```

---

## ⚠️ Exception handling en Coroutines

```kotlin
// try-catch normal dentro de coroutines
launch {
    try {
        riskyOperation()
    } catch (e: Exception) {
        handleError(e)
    }
}

// CoroutineExceptionHandler — captura excepciones no manejadas en launch
val handler = CoroutineExceptionHandler { context, exception ->
    println("Excepción en ${context[CoroutineName]}: ${exception.message}")
    logger.error("Coroutine error", exception)
}

val scope = CoroutineScope(Dispatchers.IO + handler)
scope.launch {
    throw RuntimeException("Algo salió mal")
}

// IMPORTANTE: CoroutineExceptionHandler NO captura en async — usar try-catch en await()
val deferred = scope.async {
    throw RuntimeException("Error en async")
}
try {
    deferred.await()
} catch (e: RuntimeException) {
    handleError(e)
}

// SupervisorJob — el fallo de un hijo no propaga al padre ni a los hermanos
val supervisorScope = CoroutineScope(SupervisorJob() + Dispatchers.IO + handler)

supervisorScope.launch {
    throw RuntimeException("Este fallo no afecta al hermano")
}

supervisorScope.launch {
    delay(100)
    println("Sigo corriendo después del fallo del hermano")
}

// supervisorScope {} — dentro de una coroutine
suspend fun doWorkSafely() = supervisorScope {
    val r1 = async { mayFail1() }
    val r2 = async { mayFail2() }

    val result1 = try { r1.await() } catch (e: Exception) { null }
    val result2 = try { r2.await() } catch (e: Exception) { null }

    Result(result1, result2)
}

// CancellationException — SIEMPRE relanzar
suspend fun safeWork() {
    try {
        doWork()
    } catch (e: CancellationException) {
        throw e   // ← CRÍTICO: relanzar siempre
    } catch (e: Exception) {
        handleError(e)
    }
}
```

---

## 🏗️ Structured concurrency

```kotlin
// Structured concurrency — las coroutines viven dentro de un scope
// El scope espera a todos sus hijos antes de completarse
// Si el scope se cancela, todos sus hijos se cancelan

class UserRepository(private val api: ApiService) {

    // CoroutineScope con lifecycle — se cancela cuando ya no se necesita
    private val scope = CoroutineScope(SupervisorJob() + Dispatchers.IO)

    suspend fun getUserWithPosts(userId: Long): UserWithPosts = coroutineScope {
        // Ambas peticiones corren en paralelo dentro del scope
        val user = async { api.getUser(userId) }
        val posts = async { api.getPosts(userId) }
        UserWithPosts(user.await(), posts.await())
    }

    // Cleanup — cancelar el scope cuando se destruye el repositorio
    fun clear() = scope.cancel()
}

// ViewModelScope en Android — cancela automáticamente con el ViewModel
class MyViewModel : ViewModel() {
    fun loadData() {
        viewModelScope.launch {   // Se cancela en onCleared()
            val data = repository.getData()
            _uiState.value = UiState.Success(data)
        }
    }
}

// Reglas de Structured Concurrency:
// 1. Las coroutines hijo pertenecen al scope padre
// 2. El padre espera a todos los hijos
// 3. Si el padre se cancela, se cancelan todos los hijos
// 4. Si un hijo falla, el padre y los hermanos también fallan
//    (a menos que sea SupervisorJob)

// Lanzar coroutines con scope explícito — evitar GlobalScope
fun main() = runBlocking {
    // este scope vive mientras runBlocking esté activo
    launch { task1() }
    launch { task2() }
    // runBlocking espera a que ambas terminen
}
```

---

## 🔤 String

```kotlin
// Creación
val s = "Hola mundo"
val raw = """
    Línea 1
    Línea 2
    "Con comillas"
    \n (sin escape en raw string)
""".trimIndent()   // Elimina la indentación común

// Templates
val name = "Juan"
val age = 28
"Hola $name, tenés ${age + 1} años"
"${name.uppercase()} es mayor: ${age >= 18}"
"Precio: \${price}"   // Escapar el signo $

// Propiedades
s.length
s.isEmpty()
s.isNotEmpty()
s.isBlank()         // true si vacío o solo whitespace
s.isNotBlank()

// Acceso
s[0]                // 'H' (Char)
s.first()
s.last()
s.get(0)
s.getOrNull(100)    // null si fuera de rango

// Búsqueda
s.contains("mundo")
s.contains("MUNDO", ignoreCase = true)
s.startsWith("Hola")
s.endsWith("mundo")
s.indexOf("o")          // 4
s.lastIndexOf("o")      // 9
s.indexOfFirst { it == 'o' }

// Transformación
s.uppercase()
s.lowercase()
s.capitalize()          // "Hola mundo" → "Hola mundo" (solo primera letra)
s.trim()
s.trimStart()
s.trimEnd()
s.trimIndent()          // Eliminar indentación común
s.trimMargin("|")       // Eliminar margen con prefijo
s.padStart(20)
s.padStart(20, '0')
s.padEnd(20, '-')
s.repeat(3)

// Reemplazar
s.replace("mundo", "Kotlin")
s.replace(Regex("\\s+"), "_")
s.replaceFirst("o", "0")
s.replaceRange(0, 4, "Adiós")
s.filter { it.isLetter() }   // Solo letras
s.filterNot { it.isWhitespace() }
s.map { it.uppercaseChar() }.joinToString("")

// Dividir
s.split(" ")                         // ["Hola", "mundo"]
s.split(Regex("\\s+"))
s.split(",", limit = 3)              // Máximo 3 partes
s.lines()                            // Por líneas
s.chunked(3)                         // ["Hol", "a m", "und", "o"]
s.windowed(3)                        // ["Hol", "ola", "la ", ...]
s.partition { it.isUpperCase() }     // Pair("H", "ola mundo")

// Substring
s.substring(5)          // "mundo"
s.substring(0, 4)       // "Hola"
s.substringBefore(" ")  // "Hola"
s.substringAfter(" ")   // "mundo"
s.substringBeforeLast("o")
s.substringAfterLast("o")
s.take(4)               // "Hola"
s.takeLast(5)           // "mundo"
s.drop(5)               // "mundo"
s.dropLast(5)           // "Hola "
s.removePrefix("Hola ") // "mundo"
s.removeSuffix(" mundo")// "Hola"

// Conversiones
s.toInt()
s.toIntOrNull()
s.toDouble()
s.toDoubleOrNull()
s.toBoolean()           // "true" → true, todo lo demás → false
s.toBooleanStrictOrNull() // "true"/"false" → bool, demás → null
s.toCharArray()
s.toByteArray()
s.toByteArray(Charsets.UTF_8)

// Comparación
s.compareTo("otro")
s.compareTo("otro", ignoreCase = true)
s.equals("Hola mundo", ignoreCase = true)
s.matches(Regex("\\w+"))

// Verificar contenido
s.all { it.isLetterOrDigit() }
s.any { it.isUpperCase() }
s.none { it.isDigit() }
s.count { it == 'o' }

// buildString — eficiente para concatenación
val built = buildString {
    append("Hola")
    append(", ")
    appendLine("Mundo!")
    repeat(3) { append("$it ") }
    insert(0, ">>> ")
}

// String.format
"Nombre: %s, Edad: %d, Precio: %.2f".format("Juan", 28, 99.99)
```

---

## 🔎 Regex

```kotlin
// Crear
val emailRegex = Regex("^[\\w.+-]+@[\\w-]+\\.[\\w.]+\$")
val phoneRegex = Regex("^\\+?[1-9]\\d{7,14}\$")
val uuidRegex = Regex("[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}",
    RegexOption.IGNORE_CASE)

// Options
val multiline = Regex("^inicio", setOf(RegexOption.MULTILINE))
val caseInsensitive = Regex("hola", RegexOption.IGNORE_CASE)

// Verificar
emailRegex.matches("test@email.com")           // true (match completo)
emailRegex.containsMatchIn("texto test@email.com más texto")  // true (match parcial)

// Encontrar
val match = Regex("\\d+").find("abc 123 def 456")
match?.value         // "123"
match?.range         // 4..6

val allMatches = Regex("\\d+").findAll("abc 123 def 456")
allMatches.map { it.value }.toList()   // ["123", "456"]

// Grupos
val dateRegex = Regex("(\\d{4})-(\\d{2})-(\\d{2})")
val dm = dateRegex.find("Fecha: 2026-01-15")
dm?.groupValues    // ["2026-01-15", "2026", "01", "15"]
dm?.groupValues?.get(1)  // "2026"

// Grupos nombrados
val named = Regex("(?<year>\\d{4})-(?<month>\\d{2})-(?<day>\\d{2})")
val nm = named.find("2026-01-15")
nm?.groups?.get("year")?.value   // "2026"

// Reemplazar
"Hola Mundo".replace(Regex("[aeiou]"), "*")   // "H*l* M*nd*"
"precio: 100".replace(Regex("\\d+")) { "€${it.value}" }  // "precio: €100"

// Split
"uno,dos;;tres".split(Regex("[,;]+"))   // ["uno", "dos", "tres"]

// Validadores comunes
object Validators {
    val email = Regex("^[\\w.+-]+@[\\w-]+\\.[\\w.]+\$")
    val url = Regex("^https?://[^\\s/\$.?#].[^\\s]*\$")
    val slug = Regex("^[a-z0-9]+(?:-[a-z0-9]+)*\$")
    val phone = Regex("^\\+?[1-9]\\d{7,14}\$")

    fun String.isValidEmail() = email.matches(this)
    fun String.isValidUrl() = url.matches(this)
    fun String.isValidSlug() = slug.matches(this)
}
```

---

## 🎯 Result<T>

```kotlin
// runCatching — envuelve en Result, captura cualquier excepción
val result: Result<String> = runCatching {
    riskyOperation()   // Si lanza, Result.failure; si no, Result.success
}

// Crear manualmente
val success = Result.success("valor")
val failure = Result.failure<String>(RuntimeException("Error"))

// Verificar
result.isSuccess
result.isFailure

// Obtener valor
result.getOrNull()                          // null si es failure
result.getOrDefault("por defecto")
result.getOrElse { throwable -> "Error: ${throwable.message}" }
result.getOrThrow()                         // Lanza si es failure

// Transformar
result.map { it.uppercase() }              // Si success, transforma; si failure, propaga
result.mapCatching { riskyTransform(it) }  // map que también captura excepciones
result.recover { throwable -> "recuperado" }           // Si failure, convertir en success
result.recoverCatching { throwable -> risky(throwable) } // recover que captura excepciones
result.flatMap { value -> runCatching { transform(value) } }

// Efectos secundarios
result.onSuccess { value -> println("Éxito: $value") }
result.onFailure { error -> println("Error: ${error.message}") }

// Encadenado idiomático
val finalResult = runCatching { fetchData() }
    .mapCatching { data -> parseData(data) }
    .mapCatching { parsed -> validateData(parsed) }
    .onSuccess { valid -> saveData(valid) }
    .onFailure { error -> logError(error) }

val value = finalResult.getOrElse { "valor por defecto" }

// fold — manejar ambos casos
val message = result.fold(
    onSuccess = { "Éxito: $it" },
    onFailure = { "Error: ${it.message}" }
)

// Convertir Result en sealed class propia
fun <T> Result<T>.toNetworkResult(): NetworkResult<T> = fold(
    onSuccess = { NetworkResult.Success(it) },
    onFailure = { NetworkResult.Error(it.message ?: "Error desconocido") }
)
```

---

## 🔢 Números y conversiones

```kotlin
// Conversiones entre tipos numéricos
42.toByte()
42.toShort()
42.toInt()
42.toLong()
42.toFloat()
42.toDouble()
42.toChar()      // Char con ese código ASCII/Unicode
3.14.toInt()     // 3 (trunca)
3.14f.toDouble()

// Strings ↔ números
"42".toInt()
"42".toLong()
"3.14".toDouble()
"42".toIntOrNull()     // null si falla
"abc".toLongOrNull()   // null
"0xFF".toInt(16)       // 255 — parse con base
"1010".toInt(2)        // 10 — base 2

// Formateo
42.toString()
42.toString(2)          // "101010" — binario
255.toString(16)        // "ff" — hexadecimal
3.14159.toBigDecimal().setScale(2, RoundingMode.HALF_UP)  // 3.14

// Matemáticas (kotlin.math)
import kotlin.math.*
sqrt(16.0)              // 4.0
abs(-5)                 // 5
pow(2.0, 10.0)          // 1024.0
exp(1.0)                // e
ln(E)                   // 1.0
log2(8.0)               // 3.0
log10(100.0)            // 2.0
sin(PI / 2)             // 1.0
cos(0.0)                // 1.0
tan(PI / 4)             // 1.0
floor(3.7)              // 3.0
ceil(3.2)               // 4.0
round(3.5)              // 4.0
min(3, 5)               // 3
max(3, 5)               // 5
sign(-5.0)              // -1.0
hypot(3.0, 4.0)         // 5.0

// Clamp — limitar a rango
42.coerceIn(0, 100)     // 42
150.coerceIn(0, 100)    // 100
(-5).coerceAtLeast(0)   // 0
150.coerceAtMost(100)   // 100

// BigDecimal para dinero/precisión
import java.math.BigDecimal
import java.math.RoundingMode
val price = BigDecimal("99.99")
val tax = BigDecimal("0.21")
val total = price.multiply(BigDecimal.ONE.add(tax))
    .setScale(2, RoundingMode.HALF_UP)

// Random
import kotlin.random.Random
Random.nextInt(100)            // 0..99
Random.nextInt(10, 20)         // 10..19
Random.nextDouble()            // 0.0..1.0
Random.nextDouble(1.0, 10.0)
Random.nextLong()
Random.nextBoolean()
Random.nextBytes(16)

// Semilla fija para reproducibilidad
val seeded = Random(42)
seeded.nextInt(100)
```

---

## ↕️ Comparación

```kotlin
// Comparable — ordenamiento natural
data class Person(val name: String, val age: Int) : Comparable<Person> {
    override fun compareTo(other: Person): Int = compareValuesBy(this, other,
        { it.age },          // Primero por edad
        { it.name }          // Luego por nombre
    )
}

val people = listOf(Person("Juan", 30), Person("Ana", 25), Person("Luis", 30))
people.sorted()   // [Ana(25), Juan(30), Luis(30)] — usa compareTo

// Comparator — ordenamiento alternativo
val byName = compareBy<Person> { it.name }
val byNameDesc = compareByDescending<Person> { it.name }
val byAgeAndName = compareBy<Person>({ it.age }, { it.name })

people.sortedWith(byName)
people.sortedWith(byAgeAndName)
people.maxWithOrNull(byName)

// Comparators built-in
val naturalOrder = naturalOrder<String>()
val reverseOrder = reverseOrder<String>()

listOf("b", "a", "c").sortedWith(naturalOrder)   // ["a", "b", "c"]
listOf("b", "a", "c").sortedWith(reverseOrder)   // ["c", "b", "a"]

// compareValuesBy — comparar por múltiples criterios
fun compare(a: Person, b: Person): Int =
    compareValuesBy(a, b, { it.age }, { it.name }, { it.email })

// Rangos con Comparable
data class Version(val major: Int, val minor: Int, val patch: Int) : Comparable<Version> {
    override fun compareTo(other: Version) = compareValuesBy(this, other,
        { it.major }, { it.minor }, { it.patch })
    override fun toString() = "$major.$minor.$patch"
}

val v1 = Version(1, 2, 3)
val v2 = Version(2, 0, 0)
v1 < v2    // true
v1..v2     // ClosedRange<Version> — pero no iterable sin operator iterator
v1 in Version(1, 0, 0)..Version(2, 0, 0)  // true
```

---

## 💾 kotlin.io y kotlin.system

```kotlin
import java.io.File
import kotlin.io.path.*
import kotlin.system.*

// Archivos con java.io.File + extensiones de Kotlin
val file = File("datos.txt")

// Leer
val text = file.readText()
val text2 = file.readText(Charsets.UTF_8)
val lines = file.readLines()
val bytes = file.readBytes()

// Leer línea a línea (eficiente para archivos grandes)
file.forEachLine { line -> println(line) }
file.useLines { lines -> lines.filter { it.isNotBlank() }.toList() }

// Escribir
file.writeText("contenido")
file.writeText("contenido", Charsets.UTF_8)
file.writeBytes(byteArray)
file.appendText("más contenido\n")
file.printWriter().use { out ->
    out.println("Línea 1")
    out.println("Línea 2")
}
file.bufferedWriter().use { it.write("eficiente") }

// Operaciones
file.exists()
file.isFile
file.isDirectory
file.length()               // Tamaño en bytes
file.lastModified()         // Timestamp
file.name                   // "datos.txt"
file.extension              // "txt"
file.nameWithoutExtension   // "datos"
file.parentFile             // File padre
file.absolutePath
file.canonicalPath

file.createNewFile()
file.mkdir()
file.mkdirs()               // Crear con directorios intermedios
file.delete()
file.deleteRecursively()
file.renameTo(File("nuevo.txt"))
file.copyTo(File("copia.txt"))
file.copyRecursively(File("directorio/"))

// Listar contenido de directorio
File("/tmp").listFiles()?.forEach { println(it.name) }
File("/tmp").walk().filter { it.isFile }.forEach { println(it) }
File("/tmp").walk()
    .filter { it.extension == "kt" }
    .sortedBy { it.name }
    .forEach { println(it.absolutePath) }

// Path API moderna (Java NIO + Kotlin extensions)
val path = Path("/tmp/datos.txt")
path.exists()
path.isRegularFile()
path.isDirectory()
path.readText()
path.writeText("contenido")
path.readLines()
path.forEachLine { println(it) }
path.listDirectoryEntries()
path.listDirectoryEntries("*.kt")

// stdin / stdout / stderr
val line = readLine()          // Leer una línea de stdin
print("sin salto de línea")
println("con salto de línea")
System.err.println("Error")

// measureTimeMillis — medir tiempo de ejecución
val elapsed = measureTimeMillis {
    doHeavyWork()
}
println("Tardó ${elapsed}ms")

// measureNanoTime
val nanos = measureNanoTime {
    fastOperation()
}

// exitProcess — terminar la JVM
exitProcess(0)    // Éxito
exitProcess(1)    // Error

// System.getenv / System.getProperty
val home = System.getenv("HOME")
val javaVersion = System.getProperty("java.version")
val allEnv = System.getenv()
```

---

## ☕ Interoperabilidad con Java

```kotlin
// @JvmStatic — generar método estático real en Java
class Config {
    companion object {
        @JvmStatic
        fun getInstance(): Config = Config()

        @JvmStatic
        val DEFAULT_TIMEOUT = 5000
    }
}
// En Java: Config.getInstance()  (sin .Companion)

// @JvmField — acceder como campo (no getter/setter)
class User {
    @JvmField
    val id: Long = 0L   // En Java: user.id (no user.getId())
}

// @JvmOverloads — generar sobrecargas para parámetros con default
class Connection @JvmOverloads constructor(
    val host: String,
    val port: Int = 5432,
    val timeout: Int = 30
)
// Genera en Java:
// Connection(String host)
// Connection(String host, int port)
// Connection(String host, int port, int timeout)

// @JvmName — cambiar nombre del método/archivo en bytecode
@JvmName("getProductList")
fun getProducts(): List<Product> = listOf()
// En Java: getProductList()

// @file:JvmName — cambiar nombre del archivo generado
@file:JvmName("StringUtils")
package com.example

fun String.capitalize(): String = ...
// En Java: StringUtils.capitalize(str)

// @Throws — declarar checked exceptions (para Java)
@Throws(IOException::class)
fun readFile(path: String): String = File(path).readText()

// SAM conversions — interfaces funcionales de Java como lambdas
val thread = Thread { println("En otro thread") }  // Runnable como lambda
val executor = Executor { command -> command.run() }

// SAM con interface de Kotlin — necesita fun interface
fun interface Validator<T> {
    fun validate(value: T): Boolean
}
val isPositive = Validator<Int> { it > 0 }  // SAM conversion

// Platform types — tipos de Java sin anotación de null
// Kotlin no puede saber si son null o no — se marca con !
val javaString: String! = javaObject.getString()  // Puede ser null
// Tratar como nullable para seguridad
val safe: String? = javaObject.getString()

// Acceder a clase de Java en Kotlin
val clazz = String::class.java          // java.lang.Class<String>
val kClass = String::class              // KClass<String>
val instance = clazz.getDeclaredConstructor(String::class.java).newInstance("Hola")

// Arrays de Java
val intArray: IntArray = intArrayOf(1, 2, 3)    // primitive int[]
val objArray: Array<String> = arrayOf("a", "b") // String[]

intArray.toList()
listOf(1, 2, 3).toIntArray()

// Nullability con anotaciones de Java
// @NotNull en Java → non-nullable en Kotlin
// @Nullable en Java → nullable en Kotlin
```

---

## 🛠️ build.gradle.kts

```kotlin
plugins {
    kotlin("jvm") version "2.1.0"
    kotlin("plugin.serialization") version "2.1.0"
    id("com.google.devtools.ksp") version "2.1.0-1.0.29"  // KSP para code generation
    application
}

group = "com.miempresa"
version = "1.0.0"

kotlin {
    jvmToolchain(21)   // Java 21 — más moderno que compileKotlin { kotlinOptions { jvmTarget } }
}

repositories {
    mavenCentral()
}

dependencies {
    // Coroutines
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.9.0")

    // Kotlin Serialization (alternativa a Gson/Jackson)
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.7.3")

    // Ktor — HTTP client/server puro Kotlin
    implementation("io.ktor:ktor-client-core:3.0.2")
    implementation("io.ktor:ktor-client-cio:3.0.2")
    implementation("io.ktor:ktor-client-content-negotiation:3.0.2")
    implementation("io.ktor:ktor-serialization-kotlinx-json:3.0.2")
    implementation("io.ktor:ktor-client-logging:3.0.2")

    // Arrow — programación funcional en Kotlin
    implementation("io.arrow-kt:arrow-core:1.2.4")
    implementation("io.arrow-kt:arrow-fx-coroutines:1.2.4")

    // Logging
    implementation("io.github.oshai:kotlin-logging-jvm:7.0.3")
    implementation("ch.qos.logback:logback-classic:1.5.12")

    // Testing
    testImplementation(kotlin("test"))
    testImplementation("org.jetbrains.kotlinx:kotlinx-coroutines-test:1.9.0")
    testImplementation("io.mockk:mockk:1.13.13")
    testImplementation("io.kotest:kotest-runner-junit5:5.9.1")
    testImplementation("io.kotest:kotest-assertions-core:5.9.1")
    testImplementation("io.kotest:kotest-property:5.9.1")   // Property-based testing
}

tasks.test {
    useJUnitPlatform()
}

application {
    mainClass.set("com.miempresa.MainKt")
}

// Tarea de fat JAR
tasks.jar {
    manifest { attributes["Main-Class"] = "com.miempresa.MainKt" }
    from(configurations.runtimeClasspath.get().map { if (it.isDirectory) it else zipTree(it) })
    duplicatesStrategy = DuplicatesStrategy.EXCLUDE
}
```

---

## 💡 Idioms y buenas prácticas

### Preferir sobre Java equivalente

```kotlin
// ✅ data class vs POJO con getters/setters
data class User(val id: Long, val name: String, val email: String)
// ❌ class User { private Long id; public Long getId() { return id; } ... }

// ✅ object para singletons
object Logger { fun log(msg: String) = println(msg) }
// ❌ companion object con instancia estática

// ✅ Extension functions vs utility classes
fun String.isValidEmail() = ...
// ❌ StringUtils.isValidEmail(email)

// ✅ Scope functions para inicialización
val dialog = AlertDialog().apply { title = "Hola"; message = "Mundo" }
// ❌ AlertDialog dialog = new AlertDialog(); dialog.setTitle(); dialog.setMessage();

// ✅ Elvis para null handling
val name = user?.name ?: "Anónimo"
// ❌ if (user != null && user.getName() != null) name = user.getName(); else name = "Anónimo";

// ✅ when como expresión
val result = when (type) { "A" -> 1; "B" -> 2; else -> 0 }
// ❌ switch con break y variable mutable

// ✅ let para operaciones sobre nullable
email?.let { sendEmail(it) }
// ❌ if (email != null) sendEmail(email);

// ✅ require / check / error para precondiciones
fun setAge(age: Int) { require(age >= 0) { "Edad inválida: $age" } }
// ❌ if (age < 0) throw IllegalArgumentException("Edad inválida: " + age);

// ✅ Rangos y in para verificar
if (score in 70..89) println("Bueno")
// ❌ if (score >= 70 && score <= 89)

// ✅ buildString
val msg = buildString { append("Hola "); append(name) }
// ❌ StringBuilder sb = new StringBuilder(); sb.append("Hola "); sb.append(name); sb.toString();

// ✅ Collection literals idiomáticos
val map = mapOf("key" to "value")
// ❌ Map<String, String> map = new HashMap<>(); map.put("key", "value");

// ✅ Destructuring en lambdas
map.forEach { (key, value) -> println("$key=$value") }
// ❌ map.forEach { entry -> println(entry.key + "=" + entry.value) }

// ✅ Delegation en lugar de herencia innecesaria
class LoggingList<T>(private val delegate: MutableList<T>) : MutableList<T> by delegate {
    override fun add(element: T): Boolean {
        println("Agregando: $element")
        return delegate.add(element)
    }
}
```

---

## 🎨 Patrones de diseño en Kotlin

```kotlin
// Singleton — object declaration
object Repository {
    private val cache = mutableMapOf<Long, User>()
    fun findById(id: Long) = cache[id]
}

// Factory — companion object con create()
class Connection private constructor(val host: String, val port: Int) {
    companion object {
        fun create(host: String, port: Int = 5432): Connection {
            require(port in 1..65535) { "Puerto inválido: $port" }
            return Connection(host, port)
        }
    }
}

// Builder — apply() o DSL
data class EmailBuilder(
    var to: String = "",
    var subject: String = "",
    var body: String = "",
    var cc: List<String> = emptyList()
)

fun email(block: EmailBuilder.() -> Unit) = EmailBuilder().apply(block)

val msg = email {
    to = "juan@example.com"
    subject = "Hola"
    body = "Mensaje"
}

// Observer — Flow/SharedFlow
class Store<S>(initialState: S) {
    private val _state = MutableStateFlow(initialState)
    val state: StateFlow<S> = _state.asStateFlow()

    fun dispatch(reducer: S.() -> S) {
        _state.update { it.reducer() }
    }
}

// Strategy — funciones como parámetros
fun processPayment(
    amount: Double,
    strategy: (Double) -> Boolean = ::defaultPayment
): Boolean = strategy(amount)

fun creditCardPayment(amount: Double) = amount < 10_000
fun cryptoPayment(amount: Double) = amount > 0

processPayment(500.0, ::creditCardPayment)
processPayment(1.0, ::cryptoPayment)
processPayment(200.0)  // Default

// Decorator — extension function o delegation
fun MutableList<String>.validated(): MutableList<String> {
    return object : MutableList<String> by this {
        override fun add(element: String): Boolean {
            require(element.isNotBlank()) { "No se pueden agregar strings vacíos" }
            return this@validated.add(element)
        }
    }
}

// Template Method — abstract class + lambdas
abstract class DataProcessor<T> {
    fun process(data: List<T>): List<T> {
        val filtered = filter(data)
        val transformed = transform(filtered)
        return sort(transformed)
    }

    protected abstract fun filter(data: List<T>): List<T>
    protected abstract fun transform(data: List<T>): List<T>
    protected open fun sort(data: List<T>): List<T> = data  // Default: sin ordenar
}

// Con lambdas — más flexible que herencia
fun <T> process(
    data: List<T>,
    filter: (T) -> Boolean = { true },
    transform: (T) -> T = { it },
    sort: Comparator<T>? = null
): List<T> {
    val filtered = data.filter(filter).map(transform)
    return if (sort != null) filtered.sortedWith(sort) else filtered
}

// Monad-like — encadenamiento con Result
fun validateAndProcess(input: String): Result<ProcessedData> =
    runCatching { validate(input) }
        .mapCatching { parse(it) }
        .mapCatching { enrich(it) }
        .mapCatching { save(it) }

// DSL — Domain Specific Language
class HtmlBuilder {
    private val elements = mutableListOf<String>()

    fun h1(text: String) = elements.add("<h1>$text</h1>")
    fun p(text: String) = elements.add("<p>$text</p>")
    fun ul(block: UlBuilder.() -> Unit) {
        val ul = UlBuilder().apply(block)
        elements.add(ul.build())
    }
    fun build() = "<html>${elements.joinToString("\n")}</html>"
}

class UlBuilder {
    private val items = mutableListOf<String>()
    fun li(text: String) = items.add("<li>$text</li>")
    fun build() = "<ul>${items.joinToString("")}</ul>"
}

fun html(block: HtmlBuilder.() -> Unit) = HtmlBuilder().apply(block).build()

val page = html {
    h1("Bienvenido")
    p("Este es un párrafo")
    ul {
        li("Item 1")
        li("Item 2")
        li("Item 3")
    }
}
```
