# Kotlin — Programación orientada a objetos

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
// (reemplazo propio de String.capitalize(), que es error de compilación desde Kotlin 2.1)
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
