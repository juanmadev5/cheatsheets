# Kotlin — Referencia — Idioms y patrones de diseño

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
