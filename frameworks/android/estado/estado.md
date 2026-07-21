# Android Jetpack Compose — Estado y arquitectura

---

## 🏗️ ViewModel + StateFlow

```kotlin
// ProductsViewModel.kt
class ProductsViewModel(
    private val getProductsUseCase: GetProductsUseCase,
    private val deleteProductUseCase: DeleteProductUseCase,
) : ViewModel() {

    // UI State sellado
    sealed interface UiState {
        data object Loading : UiState
        data class Success(val products: List<Product>) : UiState
        data class Error(val message: String) : UiState
    }

    // Un solo StateFlow para todo el estado
    private val _uiState = MutableStateFlow<UiState>(UiState.Loading)
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()

    // Estado de búsqueda por separado (se actualiza frecuente)
    private val _query = MutableStateFlow("")
    val query: StateFlow<String> = _query.asStateFlow()

    // Eventos de UI (one-shot: snackbars, navegación)
    private val _events = Channel<UiEvent>(Channel.BUFFERED)
    val events = _events.receiveAsFlow()

    sealed class UiEvent {
        data class ShowSnackbar(val message: String) : UiEvent()
        data class NavigateTo(val route: String) : UiEvent()
    }

    init {
        loadProducts()
        // Buscar cuando cambia el query (debounce)
        viewModelScope.launch {
            _query
                .debounce(300)
                .distinctUntilChanged()
                .collectLatest { query -> searchProducts(query) }
        }
    }

    fun loadProducts() {
        viewModelScope.launch {
            _uiState.value = UiState.Loading
            getProductsUseCase()
                .onSuccess { _uiState.value = UiState.Success(it) }
                .onFailure { _uiState.value = UiState.Error(it.message ?: "Error desconocido") }
        }
    }

    fun onQueryChange(query: String) { _query.value = query }

    fun deleteProduct(product: Product) {
        viewModelScope.launch {
            deleteProductUseCase(product.id)
                .onSuccess { _events.send(UiEvent.ShowSnackbar("Producto eliminado")) }
                .onFailure { _events.send(UiEvent.ShowSnackbar("Error al eliminar")) }
        }
    }

    private fun searchProducts(query: String) {
        viewModelScope.launch {
            getProductsUseCase(query = query)
                .onSuccess { _uiState.value = UiState.Success(it) }
                .onFailure { _uiState.value = UiState.Error(it.message ?: "Error") }
        }
    }
}

// Consumir en Composable
@Composable
fun ProductsScreen(viewModel: ProductsViewModel = koinViewModel()) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    val snackbarHostState = remember { SnackbarHostState() }

    // Eventos one-shot
    LaunchedEffect(Unit) {
        viewModel.events.collect { event ->
            when (event) {
                is UiEvent.ShowSnackbar -> snackbarHostState.showSnackbar(event.message)
                is UiEvent.NavigateTo -> { /* navController.navigate(event.route) */ }
            }
        }
    }

    Scaffold(snackbarHost = { SnackbarHost(snackbarHostState) }) { padding ->
        when (val state = uiState) {
            is UiState.Loading -> CircularProgressIndicator(Modifier.fillMaxSize().wrapContentSize())
            is UiState.Error -> ErrorView(message = state.message, onRetry = viewModel::loadProducts)
            is UiState.Success -> ProductsList(products = state.products, modifier = Modifier.padding(padding))
        }
    }
}
```

---

## 💉 Koin — Inyección de dependencias

```kotlin
// core/di/NetworkModule.kt
val networkModule = module {
    single {
        OkHttpClient.Builder()
            .addInterceptor(HttpLoggingInterceptor().apply {
                level = HttpLoggingInterceptor.Level.BODY
            })
            .connectTimeout(30, TimeUnit.SECONDS)
            .build()
    }

    single {
        Retrofit.Builder()
            .baseUrl("https://api.example.com/")
            .client(get())
            .addConverterFactory(Json.asConverterFactory("application/json".toMediaType()))
            .build()
    }

    single<ProductApiService> { get<Retrofit>().create(ProductApiService::class.java) }
}

// core/di/DatabaseModule.kt
val databaseModule = module {
    single {
        Room.databaseBuilder(get(), AppDatabase::class.java, "app_db")
            .fallbackToDestructiveMigration()
            .build()
    }

    single { get<AppDatabase>().productDao() }
    single { get<AppDatabase>().userDao() }
}

// feature/products/di/ProductsModule.kt
val productsModule = module {
    // Repositorio
    single<ProductRepository> { ProductRepositoryImpl(get(), get()) }

    // Use cases
    factory { GetProductsUseCase(get()) }
    factory { DeleteProductUseCase(get()) }

    // ViewModel — siempre viewModel { } para que Koin gestione el ciclo de vida
    viewModel { ProductsViewModel(get(), get()) }
    viewModel { params -> DetailViewModel(productId = params.get(), get()) }
}

// Obtener ViewModel en Composable
val viewModel: ProductsViewModel = koinViewModel()

// ViewModel con parámetros
val viewModel: DetailViewModel = koinViewModel(
    parameters = { parametersOf(productId) }
)

// Obtener dependencia fuera de Composable
val repo: ProductRepository by inject()           // En cualquier clase Koin-aware
val repo = KoinJavaComponent.getKoin().get<ProductRepository>() // Imperativo
```

