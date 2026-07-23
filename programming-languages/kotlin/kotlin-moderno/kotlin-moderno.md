# Kotlin — Kotlin Moderno (2.0+)

---

## 🔥 when avanzado — Kotlin 2.2+

```kotlin
// Guards en when — condición adicional después del pattern
// Preview en Kotlin 2.1 (requería opt-in), estable desde Kotlin 2.2
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

// Manejo exhaustivo con when y guard (Kotlin 2.2+)
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
