# Android Jetpack Compose — Red y APIs

---

## 🌐 Retrofit — HTTP completo

### Definición de la API

```kotlin
// ApiService.kt
interface ProductApiService {

    // GET simple
    @GET("products")
    suspend fun getProducts(): List<ProductDto>

    // GET con parámetros en la URL
    @GET("products/{id}")
    suspend fun getProductById(@Path("id") id: Int): ProductDto

    // GET con query params
    @GET("products")
    suspend fun searchProducts(
        @Query("q") query: String,
        @Query("page") page: Int = 1,
        @Query("limit") limit: Int = 20,
        @Query("category") category: String? = null,   // null → no se incluye en la URL
    ): PagedResponse<ProductDto>

    // POST con body JSON
    @POST("products")
    suspend fun createProduct(@Body product: CreateProductRequest): ProductDto

    // PUT — actualización completa
    @PUT("products/{id}")
    suspend fun updateProduct(
        @Path("id") id: Int,
        @Body product: UpdateProductRequest,
    ): ProductDto

    // PATCH — actualización parcial
    @PATCH("products/{id}")
    suspend fun patchProduct(
        @Path("id") id: Int,
        @Body fields: Map<String, @JvmSuppressWildcards Any>,
    ): ProductDto

    // DELETE
    @DELETE("products/{id}")
    suspend fun deleteProduct(@Path("id") id: Int): Response<Unit>

    // Header dinámico
    @GET("user/profile")
    suspend fun getProfile(@Header("Authorization") token: String): UserDto

    // Headers estáticos
    @Headers("Accept: application/json", "Cache-Control: no-cache")
    @GET("config")
    suspend fun getConfig(): ConfigDto

    // Subir archivo (Multipart)
    @Multipart
    @POST("upload")
    suspend fun uploadImage(
        @Part image: MultipartBody.Part,
        @Part("description") description: RequestBody,
    ): UploadResponse

    // URL dinámica completa
    @GET
    suspend fun getFromUrl(@Url url: String): ResponseBody

    // Response<T> para acceder al código HTTP
    @GET("products")
    suspend fun getProductsRaw(): Response<List<ProductDto>>
}

// Construir MultipartBody.Part para subir imagen
fun createImagePart(uri: Uri, context: Context): MultipartBody.Part {
    val bytes = context.contentResolver.openInputStream(uri)?.readBytes() ?: byteArrayOf()
    val requestBody = bytes.toRequestBody("image/jpeg".toMediaType())
    return MultipartBody.Part.createFormData("image", "photo.jpg", requestBody)
}
```

---

### DTOs con kotlinx.serialization

```kotlin
// DTO — Data Transfer Object (lo que viene de la API)
@Serializable
data class ProductDto(
    val id: Int,
    val name: String,
    val price: Double,
    @SerialName("image_url") val imageUrl: String,      // Nombre diferente en JSON
    val category: String? = null,                        // Nullable con default
    @SerialName("created_at") val createdAt: String,
    val tags: List<String> = emptyList(),
)

@Serializable
data class PagedResponse<T>(
    val data: List<T>,
    val total: Int,
    val page: Int,
    @SerialName("total_pages") val totalPages: Int,
)

@Serializable
data class CreateProductRequest(
    val name: String,
    val price: Double,
    val category: String,
)

// Polimorfismo sellado
@Serializable
sealed class ApiResult<out T> {
    @Serializable data class Success<T>(val data: T) : ApiResult<T>()
    @Serializable data class Error(val code: Int, val message: String) : ApiResult<Nothing>()
}

// Custom serializer para tipos que no soporta por defecto
object LocalDateSerializer : KSerializer<LocalDate> {
    override val descriptor = PrimitiveSerialDescriptor("LocalDate", PrimitiveKind.STRING)
    override fun serialize(encoder: Encoder, value: LocalDate) = encoder.encodeString(value.toString())
    override fun deserialize(decoder: Decoder) = LocalDate.parse(decoder.decodeString())
}

@Serializable
data class EventDto(
    val name: String,
    @Serializable(with = LocalDateSerializer::class)
    val date: LocalDate,
)
```

---

### Manejo de errores con sealed class

```kotlin
// NetworkResult — wrapper para respuestas de API
sealed class NetworkResult<out T> {
    data class Success<T>(val data: T) : NetworkResult<T>()
    data class Error(val code: Int, val message: String) : NetworkResult<Nothing>()
    data object Loading : NetworkResult<Nothing>()
    data class Exception(val e: Throwable) : NetworkResult<Nothing>()
}

// Extension para manejar llamadas de Retrofit
suspend fun <T> safeApiCall(call: suspend () -> T): NetworkResult<T> {
    return try {
        NetworkResult.Success(call())
    } catch (e: HttpException) {
        val errorBody = e.response()?.errorBody()?.string()
        NetworkResult.Error(e.code(), errorBody ?: e.message())
    } catch (e: IOException) {
        NetworkResult.Exception(e)     // Sin conexión
    } catch (e: SerializationException) {
        NetworkResult.Exception(e)     // Error de parsing
    }
}

// Uso en repositorio
class ProductRepositoryImpl(private val api: ProductApiService) : ProductRepository {
    override suspend fun getProducts(): NetworkResult<List<Product>> =
        safeApiCall { api.getProducts().map { it.toDomain() } }
}

// Uso en ViewModel
viewModelScope.launch {
    _uiState.value = UiState.Loading
    when (val result = repository.getProducts()) {
        is NetworkResult.Success -> _uiState.value = UiState.Success(result.data)
        is NetworkResult.Error -> _uiState.value = UiState.Error("Error ${result.code}: ${result.message}")
        is NetworkResult.Exception -> _uiState.value = UiState.Error(result.e.message ?: "Error de red")
        is NetworkResult.Loading -> Unit
    }
}
```

---

### OkHttp — interceptores avanzados

```kotlin
// Interceptor de autenticación con refresh token
class AuthInterceptor(
    private val tokenProvider: TokenProvider,
    private val authRepository: AuthRepository,
) : Interceptor {

    override fun intercept(chain: Interceptor.Chain): Response {
        val originalRequest = chain.request()

        // Agregar token al header
        val authenticatedRequest = originalRequest.addAuthHeader(tokenProvider.getToken())
        val response = chain.proceed(authenticatedRequest)

        // Si 401 → intentar refresh
        if (response.code == 401) {
            response.close()
            synchronized(this) {
                // Doble check para evitar múltiples refresh simultáneos
                val currentToken = tokenProvider.getToken()
                if (originalRequest.header("Authorization") != "Bearer $currentToken") {
                    return chain.proceed(originalRequest.addAuthHeader(currentToken))
                }

                val newToken = runBlocking { authRepository.refreshToken() }
                    ?: return response   // Refresh falló → forzar logout

                tokenProvider.saveToken(newToken)
                return chain.proceed(originalRequest.addAuthHeader(newToken))
            }
        }
        return response
    }

    private fun Request.addAuthHeader(token: String?): Request =
        newBuilder()
            .header("Authorization", "Bearer $token")
            .build()
}

// Interceptor de logging condicional (solo debug)
val loggingInterceptor = HttpLoggingInterceptor().apply {
    level = if (BuildConfig.DEBUG) {
        HttpLoggingInterceptor.Level.BODY
    } else {
        HttpLoggingInterceptor.Level.NONE
    }
}

// Certificate Pinning
val certificatePinner = CertificatePinner.Builder()
    .add("api.miapp.com", "sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=")
    .add("api.miapp.com", "sha256/BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=") // Backup pin
    .build()

// OkHttpClient completo
val okHttpClient = OkHttpClient.Builder()
    .addInterceptor(AuthInterceptor(tokenProvider, authRepository))
    .addInterceptor(loggingInterceptor)
    .certificatePinner(certificatePinner)
    .connectTimeout(30, TimeUnit.SECONDS)
    .readTimeout(30, TimeUnit.SECONDS)
    .writeTimeout(60, TimeUnit.SECONDS)
    .retryOnConnectionFailure(true)
    .build()

// Retrofit con kotlinx.serialization
val json = Json {
    ignoreUnknownKeys = true      // No fallar si la API agrega campos nuevos
    isLenient = true              // Ser flexible con el formato JSON
    coerceInputValues = true      // null → default value cuando el tipo no es nullable
    encodeDefaults = false        // No serializar campos con valor default
}

val retrofit = Retrofit.Builder()
    .baseUrl("https://api.miapp.com/v1/")
    .client(okHttpClient)
    .addConverterFactory(json.asConverterFactory("application/json; charset=UTF8".toMediaType()))
    .build()
```

