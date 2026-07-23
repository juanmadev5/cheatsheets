# Kotlin — Interoperabilidad con Java

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

fun String.shout(): String = "${uppercase()}!"
// En Java: StringUtils.shout(str)

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
