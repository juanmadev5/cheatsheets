# Android Jetpack Compose — Arquitectura

---

## 🏗️ Clean Architecture

```
feature/products/
├── data/
│   ├── local/
│   │   └── ProductDao.kt              # Room — ver persistencia/persistencia.md
│   ├── remote/
│   │   ├── ProductApiService.kt       # Retrofit — ver red/retrofit.md
│   │   └── dto/ProductDto.kt          # DTOs con kotlinx.serialization
│   └── repository/
│       └── ProductRepositoryImpl.kt   # Implementa el contrato del domain
├── domain/
│   ├── model/
│   │   └── Product.kt                 # Modelo de dominio puro — sin Room/Retrofit/Android
│   ├── repository/
│   │   └── ProductRepository.kt       # Interface (contrato)
│   └── usecase/
│       ├── GetProductsUseCase.kt      # Un archivo por caso de uso
│       └── DeleteProductUseCase.kt
└── presentation/
    ├── ProductsScreen.kt
    ├── ProductsViewModel.kt
    └── di/ProductsModule.kt
```

> Regla clave: `domain/` no importa nada de `data/` ni de Android — es Kotlin puro. `data/` implementa las interfaces que `domain/` define. La dependencia siempre apunta hacia adentro.

```kotlin
// domain/model/Product.kt — modelo de dominio, sin anotaciones de Room ni Retrofit
data class Product(
    val id: String,
    val name: String,
    val price: Double,
    val category: String,
)

// domain/repository/ProductRepository.kt — el contrato que domain necesita, sin implementación
interface ProductRepository {
    fun getAll(category: String? = null): Flow<List<Product>>
    suspend fun getById(id: String): Result<Product>
    suspend fun create(name: String, price: Double, category: String): Result<String>
    suspend fun delete(id: String): Result<Unit>
    suspend fun refresh(): Result<Unit>
}

// domain/usecase/GetProductsUseCase.kt — un caso de uso encapsula UNA operación de negocio
class GetProductsUseCase(private val repository: ProductRepository) {
    operator fun invoke(category: String? = null): Flow<List<Product>> =
        repository.getAll(category)
}

class DeleteProductUseCase(private val repository: ProductRepository) {
    suspend operator fun invoke(id: String): Result<Unit> = repository.delete(id)
}
```

```kotlin
// data/remote/dto/ProductDto.kt
@Serializable
data class ProductDto(
    val id: String,
    val name: String,
    val price: Double,
    val category: String,
)

fun ProductDto.toDomain() = Product(id = id, name = name, price = price, category = category)

// data/local/ProductEntity.kt — ver persistencia/persistencia.md para el DAO completo
fun ProductEntity.toDomain() = Product(id = id.toString(), name = name, price = price, category = category)
fun Product.toEntity() = ProductEntity(id = id.toIntOrNull() ?: 0, name = name, price = price, category = category)

// data/repository/ProductRepositoryImpl.kt — offline-first: Room como fuente de verdad, red para refrescar
class ProductRepositoryImpl(
    private val dao: ProductDao,
    private val api: ProductApiService,
) : ProductRepository {

    override fun getAll(category: String?): Flow<List<Product>> =
        dao.getAll()
            .map { entities -> entities.map { it.toDomain() } }
            .let { flow -> category?.let { c -> flow.map { it.filter { p -> p.category == c } } } ?: flow }

    override suspend fun getById(id: String): Result<Product> = runCatching {
        dao.getById(id.toInt())?.toDomain() ?: error("Producto no encontrado")
    }

    override suspend fun create(name: String, price: Double, category: String): Result<String> = runCatching {
        val response = api.createProduct(CreateProductRequest(name, price, category))
        dao.insert(response.toDomain().toEntity())
        response.id
    }

    override suspend fun delete(id: String): Result<Unit> = runCatching {
        api.deleteProduct(id)
        dao.deleteById(id.toInt())
    }

    override suspend fun refresh(): Result<Unit> = runCatching {
        val remote = api.getProducts()
        dao.insertAll(remote.map { it.toDomain().toEntity() })
    }
}
```

```kotlin
// presentation/ProductsViewModel.kt — el ViewModel solo conoce use cases, nunca Repository/DAO/API directo
class ProductsViewModel(
    private val getProductsUseCase: GetProductsUseCase,
    private val deleteProductUseCase: DeleteProductUseCase,
) : ViewModel() {

    val products: StateFlow<List<Product>> = getProductsUseCase()
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5_000), emptyList())

    fun deleteProduct(id: String) {
        viewModelScope.launch { deleteProductUseCase(id) }
    }
}

// presentation/di/ProductsModule.kt — Koin cablea las tres capas
val productsModule = module {
    single<ProductRepository> { ProductRepositoryImpl(get(), get()) }

    factory { GetProductsUseCase(get()) }
    factory { DeleteProductUseCase(get()) }

    viewModel { ProductsViewModel(get(), get()) }
}
```

---

## 🧪 Por qué separar en capas

| Capa | Responsabilidad | No debería conocer |
|---|---|---|
| **domain** | Reglas de negocio, modelos puros, contratos | Room, Retrofit, Compose, Android SDK |
| **data** | Implementa los contratos, mapea DTO/Entity ↔ modelo de dominio | ViewModel, Composables |
| **presentation** | Estado de UI, orquesta use cases | Detalles de Retrofit/Room (solo usa las interfaces de domain) |

- **Testear el domain sin Android**: los use cases son Kotlin puro — se testean con JUnit sin `Robolectric` ni instrumentación.
- **Cambiar de fuente de datos sin tocar el resto**: si mañana `ProductRepositoryImpl` deja de usar Room y usa DataStore, ni `domain` ni `presentation` se enteran.
- **Un caso de uso, una responsabilidad**: evita ViewModels gigantes con toda la lógica de negocio adentro.
