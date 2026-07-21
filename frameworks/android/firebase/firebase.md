# Android Jetpack Compose — Firebase

---

## 🔥 Firebase

### Setup

```kotlin
// build.gradle.kts (project)
plugins {
    id("com.google.gms.google-services") version "4.5.0" apply false
    id("com.google.firebase.crashlytics") version "3.0.2" apply false
}

// build.gradle.kts (app)
plugins {
    id("com.google.gms.google-services")
    id("com.google.firebase.crashlytics")
}

dependencies {
    // Desde el BoM 34+ los módulos KTX (firebase-auth-ktx, etc.) ya no existen:
    // las extensions de Kotlin están integradas en el módulo principal.
    val firebaseBom = platform("com.google.firebase:firebase-bom:34.16.0")
    implementation(firebaseBom)
    implementation("com.google.firebase:firebase-auth")
    implementation("com.google.firebase:firebase-firestore")
    implementation("com.google.firebase:firebase-storage")
    implementation("com.google.firebase:firebase-crashlytics")
    implementation("com.google.firebase:firebase-analytics")
    implementation("com.google.firebase:firebase-config")
    implementation("com.google.firebase:firebase-messaging")

    // Google Sign-In
    implementation("com.google.android.gms:play-services-auth:21.4.0")
    // Credential Manager (nuevo — reemplaza GoogleSignInClient)
    implementation("androidx.credentials:credentials:1.6.0")
    implementation("androidx.credentials:credentials-play-services-auth:1.6.0")
    implementation("com.google.android.libraries.identity.googleid:googleid:1.1.1")
}
```

---

### Firebase Auth

```kotlin
class AuthRepository(private val auth: FirebaseAuth = Firebase.auth) {

    val currentUser: FirebaseUser? get() = auth.currentUser
    val isLoggedIn: Boolean get() = currentUser != null

    // Estado de auth como Flow
    val authState: Flow<FirebaseUser?> = callbackFlow {
        val listener = FirebaseAuth.AuthStateListener { trySend(it.currentUser) }
        auth.addAuthStateListener(listener)
        awaitClose { auth.removeAuthStateListener(listener) }
    }

    // Registro con email y contraseña
    suspend fun register(email: String, password: String): Result<FirebaseUser> = runCatching {
        auth.createUserWithEmailAndPassword(email, password).await().user!!
    }

    // Login con email y contraseña
    suspend fun login(email: String, password: String): Result<FirebaseUser> = runCatching {
        auth.signInWithEmailAndPassword(email, password).await().user!!
    }

    // Google Sign-In con Credential Manager (moderno — 2024+)
    suspend fun signInWithGoogle(context: Context): Result<FirebaseUser> = runCatching {
        val credentialManager = CredentialManager.create(context)
        val googleIdOption = GetGoogleIdOption.Builder()
            .setFilterByAuthorizedAccounts(false)
            .setServerClientId(BuildConfig.GOOGLE_WEB_CLIENT_ID)
            .build()

        val request = GetCredentialRequest.Builder()
            .addCredentialOption(googleIdOption)
            .build()

        val result = credentialManager.getCredential(context, request)
        val googleIdToken = (result.credential as? CustomCredential)?.let {
            GoogleIdTokenCredential.createFrom(it.data).idToken
        } ?: throw Exception("No se pudo obtener token de Google")

        val firebaseCredential = GoogleAuthProvider.getCredential(googleIdToken, null)
        auth.signInWithCredential(firebaseCredential).await().user!!
    }

    // Cerrar sesión
    fun signOut() = auth.signOut()

    // Resetear contraseña
    suspend fun sendPasswordReset(email: String): Result<Unit> = runCatching {
        auth.sendPasswordResetEmail(email).await()
    }

    // Obtener ID token para enviar al backend
    suspend fun getIdToken(forceRefresh: Boolean = false): String? =
        auth.currentUser?.getIdToken(forceRefresh)?.await()?.token

    // Actualizar perfil
    suspend fun updateProfile(displayName: String, photoUrl: String? = null): Result<Unit> = runCatching {
        val request = userProfileChangeRequest {
            this.displayName = displayName
            photoUrl?.let { this.photoUri = Uri.parse(it) }
        }
        auth.currentUser!!.updateProfile(request).await()
    }
}
```

---

### Firestore — CRUD y listeners en tiempo real

```kotlin
class ProductFirestoreRepository(
    private val db: FirebaseFirestore = Firebase.firestore,
    private val auth: FirebaseAuth = Firebase.auth,
) {
    private val collection = db.collection("products")

    // Crear documento con ID automático
    suspend fun create(product: Product): Result<String> = runCatching {
        val docRef = collection.add(product.toFirestoreMap()).await()
        docRef.id
    }

    // Crear con ID específico
    suspend fun createWithId(product: Product): Result<Unit> = runCatching {
        collection.document(product.id).set(product.toFirestoreMap()).await()
    }

    // Leer documento por ID
    suspend fun getById(id: String): Result<Product?> = runCatching {
        val snapshot = collection.document(id).get().await()
        if (snapshot.exists()) snapshot.toProduct() else null
    }

    // Leer colección con filtros
    suspend fun getAll(category: String? = null): Result<List<Product>> = runCatching {
        var query: Query = collection.orderBy("createdAt", Query.Direction.DESCENDING)
        category?.let { query = query.whereEqualTo("category", it) }
        query.get().await().documents.mapNotNull { it.toProduct() }
    }

    // Actualizar campos específicos (no sobreescribe el documento entero)
    suspend fun update(id: String, fields: Map<String, Any>): Result<Unit> = runCatching {
        collection.document(id).update(fields).await()
    }

    // Eliminar
    suspend fun delete(id: String): Result<Unit> = runCatching {
        collection.document(id).delete().await()
    }

    // Listener en tiempo real — Flow (mejor que snapshots manuales)
    fun observeProducts(category: String? = null): Flow<List<Product>> = callbackFlow {
        var query: Query = collection.orderBy("createdAt", Query.Direction.DESCENDING)
        category?.let { query = query.whereEqualTo("category", it) }

        val listener = query.addSnapshotListener { snapshot, error ->
            if (error != null) {
                close(error)
                return@addSnapshotListener
            }
            val products = snapshot?.documents?.mapNotNull { it.toProduct() } ?: emptyList()
            trySend(products)
        }
        awaitClose { listener.remove() }
    }

    // Transacción — operación atómica
    suspend fun incrementStock(productId: String, amount: Int): Result<Unit> = runCatching {
        db.runTransaction { transaction ->
            val docRef = collection.document(productId)
            val snapshot = transaction.get(docRef)
            val currentStock = snapshot.getLong("stock") ?: 0
            transaction.update(docRef, "stock", currentStock + amount)
        }.await()
    }

    // Batch write — múltiples operaciones atómicas
    suspend fun deleteMultiple(ids: List<String>): Result<Unit> = runCatching {
        val batch = db.batch()
        ids.forEach { id -> batch.delete(collection.document(id)) }
        batch.commit().await()
    }

    // Paginación con cursor
    suspend fun getPage(lastDocument: DocumentSnapshot? = null, pageSize: Int = 20): Result<Pair<List<Product>, DocumentSnapshot?>> = runCatching {
        var query = collection
            .orderBy("createdAt", Query.Direction.DESCENDING)
            .limit(pageSize.toLong())
        lastDocument?.let { query = query.startAfter(it) }

        val snapshot = query.get().await()
        val products = snapshot.documents.mapNotNull { it.toProduct() }
        val lastDoc = snapshot.documents.lastOrNull()
        Pair(products, lastDoc)
    }
}

// Mappers
private fun DocumentSnapshot.toProduct(): Product? = try {
    Product(
        id = id,
        name = getString("name") ?: return null,
        price = getDouble("price") ?: 0.0,
        category = getString("category") ?: "",
        createdAt = getTimestamp("createdAt")?.toDate()?.time ?: 0L,
    )
} catch (e: Exception) { null }

private fun Product.toFirestoreMap() = mapOf(
    "name" to name,
    "price" to price,
    "category" to category,
    "createdAt" to FieldValue.serverTimestamp(),
)
```

---

### Firebase Storage

```kotlin
class StorageRepository(
    private val storage: FirebaseStorage = Firebase.storage,
) {
    // Subir imagen desde Uri
    suspend fun uploadImage(uri: Uri, path: String): Result<String> = runCatching {
        val ref = storage.reference.child(path)   // Ej: "users/uid/profile.jpg"
        ref.putFile(uri).await()
        ref.downloadUrl.await().toString()
    }

    // Subir bytes (para bitmaps procesados)
    suspend fun uploadBytes(bytes: ByteArray, path: String, mimeType: String = "image/jpeg"): Result<String> = runCatching {
        val ref = storage.reference.child(path)
        val metadata = storageMetadata { contentType = mimeType }
        ref.putBytes(bytes, metadata).await()
        ref.downloadUrl.await().toString()
    }

    // Eliminar archivo
    suspend fun delete(path: String): Result<Unit> = runCatching {
        storage.reference.child(path).delete().await()
    }

    // Obtener URL de descarga de un path existente
    suspend fun getDownloadUrl(path: String): Result<String> = runCatching {
        storage.reference.child(path).downloadUrl.await().toString()
    }
}
```

---

### Firebase Crashlytics y Remote Config

```kotlin
// Crashlytics
class CrashlyticsManager {
    private val crashlytics = Firebase.crashlytics

    fun setUser(userId: String) = crashlytics.setUserId(userId)
    fun log(message: String) = crashlytics.log(message)
    fun setCustomKey(key: String, value: String) = crashlytics.setCustomKey(key, value)
    fun recordException(e: Throwable) = crashlytics.recordException(e)

    // En el bloque catch de errores no fatales
    fun logError(context: String, e: Exception) {
        crashlytics.log("Error en: $context")
        crashlytics.recordException(e)
    }
}

// Remote Config
class RemoteConfigManager {
    private val remoteConfig = Firebase.remoteConfig

    suspend fun fetchAndActivate(): Boolean {
        remoteConfig.setDefaultsAsync(R.xml.remote_config_defaults).await()
        remoteConfig.setConfigSettingsAsync(remoteConfigSettings {
            minimumFetchIntervalInSeconds = if (BuildConfig.DEBUG) 0 else 3600
        }).await()
        return remoteConfig.fetchAndActivate().await()
    }

    fun getString(key: String) = remoteConfig.getString(key)
    fun getBoolean(key: String) = remoteConfig.getBoolean(key)
    fun getLong(key: String) = remoteConfig.getLong(key)
}
```

