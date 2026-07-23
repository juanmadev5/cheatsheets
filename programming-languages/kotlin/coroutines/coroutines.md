# Kotlin — Coroutines — Fundamentos

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
