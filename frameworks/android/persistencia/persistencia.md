# Android Jetpack Compose — Base de datos y persistencia

---

## 🗄️ Room — Base de datos local

```kotlin
// Entidad
@Entity(tableName = "products")
data class ProductEntity(
    @PrimaryKey val id: Int,
    @ColumnInfo(name = "product_name") val name: String,
    val price: Double,
    val category: String,
    @ColumnInfo(name = "created_at") val createdAt: Long = System.currentTimeMillis(),
)

// DAO
@Dao
interface ProductDao {
    @Query("SELECT * FROM products ORDER BY created_at DESC")
    fun getAll(): Flow<List<ProductEntity>>               // Flow para reactividad

    @Query("SELECT * FROM products WHERE id = :id")
    suspend fun getById(id: Int): ProductEntity?

    @Query("SELECT * FROM products WHERE product_name LIKE '%' || :query || '%'")
    fun search(query: String): Flow<List<ProductEntity>>

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(product: ProductEntity)

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertAll(products: List<ProductEntity>)

    @Update
    suspend fun update(product: ProductEntity)

    @Delete
    suspend fun delete(product: ProductEntity)

    @Query("DELETE FROM products WHERE id = :id")
    suspend fun deleteById(id: Int)

    @Query("DELETE FROM products")
    suspend fun deleteAll()

    @Transaction  // Para relaciones
    @Query("SELECT * FROM products WHERE id = :id")
    suspend fun getWithCategory(id: Int): ProductWithCategory?
}

// Relaciones
data class ProductWithCategory(
    @Embedded val product: ProductEntity,
    @Relation(
        parentColumn = "category_id",
        entityColumn = "id",
    )
    val category: CategoryEntity,
)

// Database
@Database(
    entities = [ProductEntity::class, CategoryEntity::class],
    version = 2,
    exportSchema = true,
)
@TypeConverters(Converters::class)
abstract class AppDatabase : RoomDatabase() {
    abstract fun productDao(): ProductDao
    abstract fun categoryDao(): CategoryDao

    companion object {
        val MIGRATION_1_2 = object : Migration(1, 2) {
            override fun migrate(db: SupportSQLiteDatabase) {
                db.execSQL("ALTER TABLE products ADD COLUMN category TEXT NOT NULL DEFAULT ''")
            }
        }
    }
}

// TypeConverter para tipos complejos
class Converters {
    @TypeConverter fun fromList(list: List<String>): String = Json.encodeToString(list)
    @TypeConverter fun toList(value: String): List<String> = Json.decodeFromString(value)
}

// Repositorio con Room
class ProductRepositoryImpl(
    private val dao: ProductDao,
    private val api: ProductApiService,
) : ProductRepository {

    override fun getProducts(): Flow<List<Product>> =
        dao.getAll().map { entities -> entities.map { it.toDomain() } }

    override suspend fun refreshProducts() {
        val remote = api.getProducts()
        dao.insertAll(remote.map { it.toEntity() })
    }
}
```

---

## 💾 DataStore — Preferencias persistentes

```kotlin
// DataStoreManager.kt
private val Context.dataStore by preferencesDataStore(name = "user_prefs")

class DataStoreManager(private val context: Context) {

    companion object {
        val IS_DARK_MODE = booleanPreferencesKey("is_dark_mode")
        val AUTH_TOKEN = stringPreferencesKey("auth_token")
        val USER_ID = intPreferencesKey("user_id")
        val ONBOARDING_DONE = booleanPreferencesKey("onboarding_done")
    }

    // Leer preferencia como Flow
    val isDarkMode: Flow<Boolean> = context.dataStore.data
        .catch { if (it is IOException) emit(emptyPreferences()) else throw it }
        .map { prefs -> prefs[IS_DARK_MODE] ?: false }

    val authToken: Flow<String?> = context.dataStore.data
        .catch { if (it is IOException) emit(emptyPreferences()) else throw it }
        .map { prefs -> prefs[AUTH_TOKEN] }

    // Escribir preferencias
    suspend fun setDarkMode(enabled: Boolean) {
        context.dataStore.edit { prefs -> prefs[IS_DARK_MODE] = enabled }
    }

    suspend fun saveAuthToken(token: String) {
        context.dataStore.edit { prefs -> prefs[AUTH_TOKEN] = token }
    }

    suspend fun clearSession() {
        context.dataStore.edit { prefs ->
            prefs.remove(AUTH_TOKEN)
            prefs.remove(USER_ID)
        }
    }
}

// Módulo Koin
val datastoreModule = module {
    single { DataStoreManager(get()) }
}

// Consumir en ViewModel
class SettingsViewModel(private val dataStore: DataStoreManager) : ViewModel() {
    val isDarkMode = dataStore.isDarkMode.stateIn(
        scope = viewModelScope,
        started = SharingStarted.WhileSubscribed(5_000),
        initialValue = false,
    )

    fun toggleDarkMode(enabled: Boolean) {
        viewModelScope.launch { dataStore.setDarkMode(enabled) }
    }
}
```

---

## 📄 Paging 3

```kotlin
// PagingSource — fuente de datos paginada
class ProductPagingSource(
    private val api: ProductApiService,
    private val query: String,
) : PagingSource<Int, Product>() {

    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, Product> {
        return try {
            val page = params.key ?: 1
            val response = api.searchProducts(query = query, page = page, pageSize = params.loadSize)
            LoadResult.Page(
                data = response.products.map { it.toDomain() },
                prevKey = if (page == 1) null else page - 1,
                nextKey = if (response.products.isEmpty()) null else page + 1,
            )
        } catch (e: Exception) {
            LoadResult.Error(e)
        }
    }

    override fun getRefreshKey(state: PagingState<Int, Product>): Int? =
        state.anchorPosition?.let { anchor ->
            state.closestPageToPosition(anchor)?.prevKey?.plus(1)
                ?: state.closestPageToPosition(anchor)?.nextKey?.minus(1)
        }
}

// RemoteMediator — Room + API (offline-first con paginación)
@OptIn(ExperimentalPagingApi::class)
class ProductRemoteMediator(
    private val api: ProductApiService,
    private val db: AppDatabase,
) : RemoteMediator<Int, ProductEntity>() {

    override suspend fun load(
        loadType: LoadType,
        state: PagingState<Int, ProductEntity>,
    ): MediatorResult {
        return try {
            val page = when (loadType) {
                LoadType.REFRESH -> 1
                LoadType.PREPEND -> return MediatorResult.Success(endOfPaginationReached = true)
                LoadType.APPEND -> {
                    val lastItem = state.lastItemOrNull()
                        ?: return MediatorResult.Success(endOfPaginationReached = false)
                    (lastItem.id / state.config.pageSize) + 1
                }
            }

            val response = api.getProducts(page = page, pageSize = state.config.pageSize)

            db.withTransaction {
                if (loadType == LoadType.REFRESH) db.productDao().deleteAll()
                db.productDao().insertAll(response.products.map { it.toEntity() })
            }

            MediatorResult.Success(endOfPaginationReached = response.products.isEmpty())
        } catch (e: Exception) {
            MediatorResult.Error(e)
        }
    }
}

// Repositorio con Paging
class ProductRepositoryImpl(
    private val api: ProductApiService,
    private val db: AppDatabase,
) : ProductRepository {

    @OptIn(ExperimentalPagingApi::class)
    override fun getPagedProducts(): Flow<PagingData<Product>> = Pager(
        config = PagingConfig(pageSize = 20, enablePlaceholders = false),
        remoteMediator = ProductRemoteMediator(api, db),
        pagingSourceFactory = { db.productDao().getPagingSource() },
    ).flow.map { pagingData -> pagingData.map { it.toDomain() } }
}

// ViewModel
class ProductsViewModel(private val repo: ProductRepository) : ViewModel() {
    val products: Flow<PagingData<Product>> = repo.getPagedProducts()
        .cachedIn(viewModelScope)   // cachedIn es CRUCIAL para sobrevivir rotaciones
}

// Composable con LazyPagingItems
@Composable
fun ProductsScreen(viewModel: ProductsViewModel = koinViewModel()) {
    val products = viewModel.products.collectAsLazyPagingItems()

    LazyColumn {
        items(count = products.itemCount, key = products.itemKey { it.id }) { index ->
            val product = products[index]
            if (product != null) ProductCard(product)
            else ShimmerCard()
        }

        // Estados de carga
        when (val state = products.loadState.append) {
            is LoadState.Loading -> item { CircularProgressIndicator(Modifier.fillMaxWidth().wrapContentSize()) }
            is LoadState.Error -> item {
                Text("Error: ${state.error.message}")
                Button(onClick = { products.retry() }) { Text("Reintentar") }
            }
            is LoadState.NotLoading -> Unit
        }
    }

    // Refresh
    when (val state = products.loadState.refresh) {
        is LoadState.Loading -> Box(Modifier.fillMaxSize()) {
            CircularProgressIndicator(Modifier.align(Alignment.Center))
        }
        is LoadState.Error -> ErrorView(
            message = state.error.message ?: "Error",
            onRetry = { products.refresh() },
        )
        else -> Unit
    }
}
```

