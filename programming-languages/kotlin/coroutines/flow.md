# Kotlin — Coroutines — Flow, StateFlow, SharedFlow y Channel

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
