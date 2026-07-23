# Kotlin — Standard Library

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
// capitalize() es error de compilación desde Kotlin 2.1 — usar replaceFirstChar
s.replaceFirstChar { if (it.isLowerCase()) it.titlecase() else it.toString() }
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
