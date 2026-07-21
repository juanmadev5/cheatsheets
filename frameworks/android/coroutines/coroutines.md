# Android Jetpack Compose — Coroutines y Flow

---

## ⚡ Coroutines & Flow

### Conceptos base

```kotlin
// CoroutineScope y Dispatchers
viewModelScope.launch { }                          // Main thread, cancelado con el ViewModel
lifecycleScope.launch { }                          // Main thread, cancelado con el Lifecycle
lifecycleScope.launch(Dispatchers.IO) { }          // IO thread (red, disco)
lifecycleScope.launch(Dispatchers.Default) { }     // CPU intensivo (sorting, parsing)
CoroutineScope(Dispatchers.IO).launch { }          // Scope manual (requiere cancelación manual)

// withContext — cambiar de dispatcher dentro de una coroutine
suspend fun loadData(): List<Item> = withContext(Dispatchers.IO) {
    api.fetchItems()   // Corre en IO
}                      // Automáticamente vuelve al dispatcher original

// async / await — paralelismo
val result = viewModelScope.async {
    val users = async { api.fetchUsers() }
    val products = async { api.fetchProducts() }
    Pair(users.await(), products.await())   // Ambas llamadas corren en paralelo
}.await()

// launch vs async
// launch → fire-and-forget, devuelve Job
// async  → devuelve Deferred<T>, se espera con .await()

// supervisorScope — una falla no cancela las demás
supervisorScope {
    val job1 = launch { riskyOperation1() }
    val job2 = launch { riskyOperation2() }   // Si job1 falla, job2 sigue
}

// coroutineScope — una falla cancela todas
coroutineScope {
    val r1 = async { operation1() }
    val r2 = async { operation2() }
    r1.await() + r2.await()
}

// Timeout
val result = withTimeoutOrNull(5_000L) {
    api.fetchData()   // null si supera 5 segundos
}

// Cancelación y cleanup
val job = viewModelScope.launch {
    try {
        doWork()
    } catch (e: CancellationException) {
        // Siempre relanzar CancellationException
        throw e
    } finally {
        // Limpieza garantizada
        cleanup()
    }
}
job.cancel()
job.cancelAndJoin()   // Cancela y espera a que termine
```

---

### Flow — streams reactivos

```kotlin
// Flow básico
fun numbersFlow(): Flow<Int> = flow {
    for (i in 1..5) {
        delay(100)
        emit(i)
    }
}

// Collectar un Flow
viewModelScope.launch {
    numbersFlow().collect { value -> println(value) }
}

// Operadores esenciales
flow
    .map { it * 2 }                          // Transformar cada valor
    .filter { it > 4 }                       // Filtrar valores
    .take(3)                                 // Tomar solo los primeros N
    .drop(2)                                 // Descartar los primeros N
    .distinctUntilChanged()                  // Emitir solo si cambió el valor
    .debounce(300)                           // Esperar 300ms de silencio antes de emitir
    .sample(1_000)                           // Emitir el último valor cada 1000ms (throttle)
    .onEach { log(it) }                      // Efecto secundario sin transformar
    .catch { e -> emit(defaultValue) }       // Manejo de errores
    .onStart { emit(loadingValue) }          // Emitir antes de que empiece
    .onCompletion { emit(doneValue) }        // Emitir al finalizar
    .flowOn(Dispatchers.IO)                  // Cambiar dispatcher del productor

// combine — combinar múltiples flows en uno
combine(flowA, flowB, flowC) { a, b, c ->
    "$a $b $c"
}.collect { combined -> }

// zip — combinar par a par (espera ambos)
flowA.zip(flowB) { a, b -> Pair(a, b) }.collect { }

// flatMapLatest — cancelar el anterior al llegar uno nuevo
searchQuery
    .debounce(300)
    .flatMapLatest { query -> api.search(query) }
    .collect { results -> }

// flatMapMerge — no cancela, corre en paralelo
// flatMapConcat — espera cada uno (secuencial)

// collectLatest — cancela el procesamiento anterior
flow.collectLatest { value ->
    delay(1000)        // Si llega un nuevo valor, esta ejecución se cancela
    process(value)
}
```

---

### StateFlow vs SharedFlow

```kotlin
// StateFlow — estado con valor actual (siempre tiene un valor)
// - Siempre emite el último valor a nuevos suscriptores
// - Solo emite si el valor cambió (distinctUntilChanged implícito)
// - Ideal para: estado de UI, datos de pantalla
private val _uiState = MutableStateFlow(UiState.Loading)
val uiState: StateFlow<UiState> = _uiState.asStateFlow()

_uiState.value = UiState.Success(data)      // Desde cualquier thread
_uiState.update { current -> current.copy(isLoading = false) }  // Update atómico

// SharedFlow — eventos de difusión (sin valor inicial)
// - Puede tener múltiples suscriptores
// - No guarda valor (a menos que configures replay)
// - Ideal para: eventos one-shot (snackbar, navegación), broadcasts
private val _events = MutableSharedFlow<UiEvent>(
    replay = 0,             // Sin replay (default)
    extraBufferCapacity = 16,
    onBufferOverflow = BufferOverflow.DROP_OLDEST,
)
val events = _events.asSharedFlow()

viewModelScope.launch { _events.emit(UiEvent.ShowSnackbar("Guardado")) }

// Channel — para comunicación punto a punto (un productor, un consumidor)
// Ideal para: eventos UI en ViewModel → Composable
private val _channel = Channel<UiEvent>(Channel.BUFFERED)
val events = _channel.receiveAsFlow()

viewModelScope.launch { _channel.send(UiEvent.NavigateToDetail(id)) }

// stateIn — convertir Flow a StateFlow
val products: StateFlow<List<Product>> = repository.getProducts()
    .stateIn(
        scope = viewModelScope,
        started = SharingStarted.WhileSubscribed(5_000),  // Cancelar si no hay suscriptores por 5s
        initialValue = emptyList(),
    )

// SharingStarted.Eagerly       — empieza de inmediato, nunca para
// SharingStarted.Lazily        — empieza al primer suscriptor, nunca para
// SharingStarted.WhileSubscribed(5_000) — producción para si no hay suscriptores (ahorra recursos)

// snapshotFlow — convertir estado Compose a Flow
LaunchedEffect(Unit) {
    snapshotFlow { scrollState.firstVisibleItemIndex }
        .filter { it > 10 }
        .collect { viewModel.loadMoreItems() }
}

// callbackFlow — convertir callbacks a Flow
fun locationFlow(client: FusedLocationProviderClient): Flow<Location> = callbackFlow {
    val callback = object : LocationCallback() {
        override fun onLocationResult(result: LocationResult) {
            result.lastLocation?.let { trySend(it) }
        }
    }
    client.requestLocationUpdates(locationRequest, callback, Looper.getMainLooper())
    awaitClose { client.removeLocationUpdates(callback) }
}
```

