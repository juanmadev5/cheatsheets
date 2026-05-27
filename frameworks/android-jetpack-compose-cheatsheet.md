# Android Jetpack Compose — Cheatsheet

---

## 📑 Índice

### Base
- [Gradle — dependencias esenciales](#️-gradle--dependencias-esenciales)
- [Estructura de proyecto](#-estructura-de-proyecto)
- [Punto de entrada — App, MainActivity, Koin init](#-punto-de-entrada)

### Composables y UI
- [Composables fundamentales — Stateless vs Stateful, state hoisting](#-composables-fundamentales)
- [Layout — Column, Row, Box, LazyColumn, LazyRow](#-layout)
- [Modifier — operaciones comunes](#modifier--operaciones-comunes)
- [Widgets visuales — Text, AsyncImage, Botones, TextField, Form](#️-widgets-visuales-comunes)

### UI avanzada
- [PullToRefresh](#pulltorefresh)
- [SwipeToDismiss](#swipetodismiss)
- [ModalBottomSheet, DatePicker, TimePicker, SearchBar, ExposedDropdown, StickyHeader](#modalBottomsheet-datepicker-y-searchbar)
- [Animaciones — AnimatedVisibility, animateAsState, Crossfade, Transition, Infinite](#-animaciones)

### Navegación
- [Navigation 3 — conceptos, back stack, entryProvider, NavDisplay](#-navigation-3-nuevo-sistema--recomendado)
- [Navigation 3 — operaciones sobre el back stack](#operaciones-sobre-el-back-stack)
- [Navigation 3 — ViewModel con scope por NavEntry](#viewmodel-con-scope-por-naventry)
- [Navigation 3 — BottomBar](#bottombar-con-navigation-3)
- [Navigation 3 — navegación condicional (auth guard)](#navegación-condicional-auth-guard)
- [Navigation 2 — legacy, type-safe](#-navigation-2-legacy--referencia)

### Estado y arquitectura
- [ViewModel + StateFlow — UiState sellado, Channel, eventos one-shot](#️-viewmodel--stateflow)
- [Koin — módulos, viewModel, factory, single, WorkManager](#-koin--inyección-de-dependencias)
- [Clean Architecture — repositorio, use cases, get_it](#️-clean-architecture) *(ver sección Coroutines)*

### Base de datos y persistencia
- [Room — entidades, DAO, relaciones, TypeConverter, migraciones](#️-room--base-de-datos-local)
- [DataStore — leer/escribir preferencias, stateIn](#-datastore--preferencias-persistentes)
- [Paging 3 — PagingSource, RemoteMediator, cachedIn, LazyPagingItems](#-paging-3)

### Red y APIs
- [Retrofit — @GET/POST/PUT/PATCH/DELETE, @Multipart, Response<T>](#-retrofit--http-completo)
- [DTOs con kotlinx.serialization — @SerialName, polimorfismo, custom serializer](#dtos-con-kotlinxserialization)
- [Manejo de errores — NetworkResult, safeApiCall](#manejo-de-errores-con-sealed-class)
- [OkHttp — AuthInterceptor con refresh token, certificate pinning](#okhttp--interceptores-avanzados)

### Coroutines y Flow
- [Coroutines — launch, async, withContext, supervisorScope, timeout, cancelación](#-coroutines--flow)
- [Flow — operadores, collect, flatMapLatest, combine, zip, collectLatest](#flow--streams-reactivos)
- [StateFlow vs SharedFlow vs Channel — cuándo usar cada uno, stateIn, snapshotFlow, callbackFlow](#stateflow-vs-sharedflow)

### Firebase
- [Setup — BOM, plugins, google-services](#-firebase)
- [Firebase Auth — email, Google (Credential Manager), biométrico, ID token](#firebase-auth)
- [Firestore — CRUD, Flow en tiempo real, transacciones, batch, paginación con cursor](#firestore--crud-y-listeners-en-tiempo-real)
- [Firebase Storage](#firebase-storage)
- [Crashlytics y Remote Config](#firebase-crashlytics-y-remote-config)

### Seguridad
- [EncryptedSharedPreferences y Keystore](#encryptedsharedpreferences-y-keystore)
- [BiometricPrompt — huella, face, PIN fallback](#biometricprompt--huella-digital-y-reconocimiento-facial)
- [Network Security Config — HTTPS, certificate pinning, debug overrides](#network-security-config--https-y-certificate-pinning)

### Background y servicios
- [WorkManager — CoroutineWorker, Koin, OneTime, Periodic, encadenado, constraints](#️-workmanager)
- [Foreground Service — MediaPlayback, binder, START_NOT_STICKY](#-foreground-service)
- [Broadcast Receivers — dinámico, callbackFlow, LocalBroadcastManager](#-broadcast-receivers)

### Notificaciones
- [Notificaciones — canales, PendingIntent, acciones, badges](#-notificaciones)
- [Notificaciones y alarmas — permisos Android 13+](#-notificaciones-y-alarmas)

### Ciclo de vida y estado
- [rememberSaveable — Saver personalizado](#remembersaveable-y-savedstatehandle)
- [SavedStateHandle — StateFlow, parámetros de navegación](#remembersaveable-y-savedstatehandle)
- [BackHandler — interceptar botón atrás con diálogo](#remembersaveable-y-savedstatehandle)

### Permisos y Manifest
- [AndroidManifest.xml — estructura básica](#-androidmanifestxml--permisos-completos)
- [Tipos de permisos — normal, dangerous, signature](#tipos-de-permisos-según-su-nivel-de-protección)
- [Red y conectividad — permisos](#-red-y-conectividad)
- [Ubicación — coarse, fine, background](#-ubicación)
- [Cámara y multimedia — Android 13+ READ_MEDIA_*](#️-cámara-y-multimedia)
- [Contactos y calendarios](#-contactos-y-calendarios)
- [Teléfono y SMS](#-teléfono-y-sms)
- [Notificaciones y alarmas — POST_NOTIFICATIONS, SCHEDULE_EXACT_ALARM](#-notificaciones-y-alarmas)
- [Batería y background — WAKE_LOCK, FOREGROUND_SERVICE types](#️-batería-y-background)
- [Sensores y actividad física — ACTIVITY_RECOGNITION](#️-sensores-y-actividad-física)
- [Permisos especiales — SYSTEM_ALERT_WINDOW, MANAGE_EXTERNAL_STORAGE](#-permisos-especiales-requieren-intent-a-ajustes)
- [Pedir permisos en runtime — RequestPermission, RequestMultiplePermissions](#-pedir-permisos-en-runtime-compose)
- [uses-feature — declarar hardware requerido vs opcional](#-uses-feature--declarar-hardware-requerido)

### Intents
- [Tipos de Intent — explícito, implícito, broadcast, pending](#-intents--comunicación-entre-componentes)
- [Intent explícito — Activity, extras, Parcelable, ActivityResultContracts](#intent-explícito)
- [Intent implícito — URL, dialer, email, compartir, maps, ajustes](#intent-implícito--acciones-del-sistema)
- [Intent Flags — NEW_TASK, CLEAR_TASK, SINGLE_TOP, GRANT_READ_URI](#intent-flags-importantes)
- [PendingIntent — notificaciones, AlarmManager, FLAG_IMMUTABLE](#pendingintent--para-notificaciones-alarmas-y-widgets)
- [Deep Links — intent-filter, esquemas, leer URI](#deep-links--recibir-intents-en-tu-app)

### Almacenamiento
- [Resumen de opciones — internal, external, MediaStore, SAF, Downloads](#-almacenamiento--leer-y-escribir-en-directorios-públicos)
- [MediaStore — escribir imágenes, videos, Downloads con IS_PENDING](#mediastore--escribir-en-directorios-públicos-imágenes-video-audio)
- [MediaStore — leer con query, proyección, filtros](#mediastore--leer-archivos-públicos)
- [SAF — OpenDocument, CreateDocument, OpenDocumentTree, permisos persistentes](#saf-storage-access-framework--acceso-a-cualquier-directorio)
- [FileProvider — compartir archivos internos, capturar foto con cámara](#fileprovider--compartir-archivos-internos-con-otras-apps)
- [Almacenamiento interno — openFileOutput, filesDir, cacheDir](#almacenamiento-interno--archivos-privados-de-la-app)
- [Almacenamiento externo privado — getExternalFilesDir, sin permisos Android 10+](#almacenamiento-externo-privado-sin-permisos-en-android-10)
- [MANAGE_EXTERNAL_STORAGE — para file managers](#acceso-completo-al-almacenamiento-manage_external_storage)
- [MIME types comunes](#mime-types-comunes)

### Temas
- [Temas con Material 3 — lightColorScheme, darkColorScheme, Dynamic Color](#-temas-con-material-3)

### Performance
- [Compose performance — derivedStateOf, @Stable/@Immutable, CompositionLocal, key(), lambdas estabilizadas](#-compose--performance-y-recomposición)

### Accesibilidad
- [Accesibilidad — semantics, mergeDescendants, CustomAction, LiveRegion, heading](#️-accesibilidad)

### Internacionalización
- [Localización — strings.xml, plurales, formateo con locale, idioma en runtime, RTL](#-localización-e-internacionalización)

### Widgets de pantalla de inicio
- [App Widget con Glance — GlanceAppWidget, ActionCallback, appwidget-provider](#-app-widget-con-glance)

### Connectivity
- [Connectivity — NetworkCallback como Flow, banner offline, retry automático](#-connectivity--monitoreo-de-red)

### Play Store
- [In-App Updates — flexible, inmediata, prioridades, InstallStateListener](#-in-app-updates-e-in-app-reviews)
- [In-App Reviews — requestReviewFlow, launchReviewFlow](#-in-app-updates-e-in-app-reviews)

### Build y distribución
- [Build Variants y Flavors — signingConfigs, buildTypes, productFlavors, BuildConfig](#️-build-variants-y-flavors)
- [ProGuard / R8 — reglas para Kotlin, Retrofit, Room, Koin, Firebase, Parcelable](#️-proguard--r8--reglas-esenciales)

### Referencia
- [Paquetes y librerías esenciales](#-paquetes-y-librerías-esenciales)
- [Buenas prácticas](#-buenas-prácticas)

---

## ⚙️ Gradle — dependencias esenciales

```kotlin
// build.gradle.kts (app)
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.compose)
    alias(libs.plugins.ksp)                    // KSP para Room, Koin annotations
    alias(libs.plugins.kotlin.serialization)
}

android {
    compileSdk = 35
    defaultConfig {
        minSdk = 26
        targetSdk = 35
    }
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
    kotlinOptions { jvmTarget = "17" }
    buildFeatures { compose = true }
}

dependencies {
    // Compose BOM — versiona todo Compose automáticamente
    val composeBom = platform("androidx.compose:compose-bom:2024.12.01")
    implementation(composeBom)
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.ui:ui-tooling-preview")
    implementation("androidx.compose.material3:material3")
    debugImplementation("androidx.compose.ui:ui-tooling")

    // Activity & Lifecycle
    implementation("androidx.activity:activity-compose:1.9.3")
    implementation("androidx.lifecycle:lifecycle-runtime-compose:2.8.7")
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.8.7")

    // Navigation
    implementation("androidx.navigation:navigation-compose:2.8.4")

    // Room
    implementation("androidx.room:room-runtime:2.6.1")
    implementation("androidx.room:room-ktx:2.6.1")
    ksp("androidx.room:room-compiler:2.6.1")

    // DataStore
    implementation("androidx.datastore:datastore-preferences:1.1.1")

    // WorkManager
    implementation("androidx.work:work-runtime-ktx:2.10.0")

    // Paging 3
    implementation("androidx.paging:paging-runtime-ktx:3.3.4")
    implementation("androidx.paging:paging-compose:3.3.4")

    // Koin
    implementation("io.insert-koin:koin-android:3.5.6")
    implementation("io.insert-koin:koin-androidx-compose:3.5.6")
    implementation("io.insert-koin:koin-androidx-workmanager:3.5.6")

    // Retrofit + OkHttp
    implementation("com.squareup.retrofit2:retrofit:2.11.0")
    implementation("com.squareup.retrofit2:converter-kotlinx-serialization:2.11.0")
    implementation("com.squareup.okhttp3:logging-interceptor:4.12.0")

    // Kotlin Serialization
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.7.3")

    // Coroutines
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.8.1")

    // Coil — imágenes
    implementation("io.coil-kt:coil-compose:2.7.0")
}
```

---

## 📁 Estructura de proyecto

```
app/src/main/
├── java/com/empresa/app/
│   ├── App.kt                          # Application class (Koin init)
│   ├── MainActivity.kt
│   ├── core/
│   │   ├── di/                         # Módulos Koin globales
│   │   ├── network/                    # Retrofit, OkHttp
│   │   ├── database/                   # Room Database
│   │   └── datastore/                  # DataStore preferences
│   └── feature/
│       └── products/
│           ├── data/
│           │   ├── local/              # DAO, entidades Room
│           │   ├── remote/             # API service, DTOs
│           │   └── repository/         # Implementación del repo
│           ├── domain/
│           │   ├── model/              # Modelos de dominio
│           │   ├── repository/         # Interface del repo
│           │   └── usecase/            # Casos de uso
│           └── presentation/
│               ├── ProductsScreen.kt
│               ├── ProductsViewModel.kt
│               └── di/                 # Módulo Koin del feature
└── res/
    ├── values/
    │   ├── strings.xml
    │   └── themes.xml
    └── xml/
        └── backup_rules.xml
```

---

## 🚀 Punto de entrada

```kotlin
// App.kt
class App : Application() {
    override fun onCreate() {
        super.onCreate()
        startKoin {
            androidLogger(Level.DEBUG)
            androidContext(this@App)
            workManagerFactory()          // Para Workers con Koin
            modules(
                networkModule,
                databaseModule,
                datastoreModule,
                productsModule,
            )
        }
    }
}

// AndroidManifest.xml
// <application android:name=".App" ...>

// MainActivity.kt
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            MyAppTheme {
                AppNavGraph()
            }
        }
    }
}
```

---

## 🧱 Composables fundamentales

### Stateless vs Stateful

```kotlin
// Stateless — recibe todo por parámetros (preferido para UI pura)
@Composable
fun UserCard(
    user: User,
    onDelete: () -> Unit,
    modifier: Modifier = Modifier,   // Siempre incluir modifier
) {
    Card(modifier = modifier.fillMaxWidth()) {
        Text(text = user.name)
        IconButton(onClick = onDelete) {
            Icon(Icons.Default.Delete, contentDescription = "Eliminar")
        }
    }
}

// Stateful — gestiona su propio estado (usar lo mínimo posible)
@Composable
fun ExpandableCard(title: String) {
    var expanded by remember { mutableStateOf(false) }

    Card(onClick = { expanded = !expanded }) {
        Text(title)
        AnimatedVisibility(visible = expanded) {
            Text("Contenido expandido...")
        }
    }
}
```

### State hoisting

```kotlin
// Patrón: estado arriba, eventos hacia abajo
@Composable
fun SearchBar(
    query: String,                       // Estado bajando
    onQueryChange: (String) -> Unit,     // Evento subiendo
    modifier: Modifier = Modifier,
) {
    TextField(
        value = query,
        onValueChange = onQueryChange,
        modifier = modifier,
        placeholder = { Text("Buscar...") },
    )
}

// El padre gestiona el estado
@Composable
fun SearchScreen(viewModel: SearchViewModel = koinViewModel()) {
    val query by viewModel.query.collectAsStateWithLifecycle()

    SearchBar(
        query = query,
        onQueryChange = viewModel::onQueryChange,
    )
}
```

---

## 📐 Layout

```kotlin
// Column
Column(
    modifier = Modifier
        .fillMaxSize()
        .padding(16.dp),
    verticalArrangement = Arrangement.spacedBy(8.dp),
    horizontalAlignment = Alignment.CenterHorizontally,
) {
    Text("Primero")
    Text("Segundo")
}

// Row
Row(
    modifier = Modifier.fillMaxWidth(),
    horizontalArrangement = Arrangement.SpaceBetween,
    verticalAlignment = Alignment.CenterVertically,
) {
    Icon(Icons.Default.Home, contentDescription = null)
    Text("Título")
    Icon(Icons.Default.Settings, contentDescription = null)
}

// Box (equivalente a FrameLayout / Stack)
Box(modifier = Modifier.fillMaxSize()) {
    Image(painter = painterResource(R.drawable.bg), contentDescription = null,
        modifier = Modifier.fillMaxSize(), contentScale = ContentScale.Crop)
    Text("Encima", modifier = Modifier.align(Alignment.Center))
    FloatingActionButton(
        onClick = {},
        modifier = Modifier.align(Alignment.BottomEnd).padding(16.dp),
    ) { Icon(Icons.Default.Add, contentDescription = "Agregar") }
}

// LazyColumn — lista eficiente (equivalente a RecyclerView)
LazyColumn(
    contentPadding = PaddingValues(16.dp),
    verticalArrangement = Arrangement.spacedBy(8.dp),
) {
    item { Text("Header") }

    items(items = products, key = { it.id }) { product ->
        ProductCard(product = product)
    }

    itemsIndexed(items = products) { index, product ->
        Text("$index: ${product.name}")
    }

    item { Text("Footer") }
}

// LazyRow — lista horizontal
LazyRow(horizontalArrangement = Arrangement.spacedBy(8.dp)) {
    items(categories) { category ->
        FilterChip(selected = false, onClick = {}, label = { Text(category) })
    }
}
```

### Modifier — operaciones comunes

```kotlin
Modifier
    .fillMaxSize()                              // Ocupa todo el espacio disponible
    .fillMaxWidth()                             // Solo ancho completo
    .fillMaxHeight(0.5f)                        // 50% del alto
    .size(48.dp)                                // Tamaño fijo
    .width(200.dp).height(100.dp)
    .wrapContentSize()                          // Solo el tamaño necesario
    .padding(16.dp)                             // Padding uniforme
    .padding(horizontal = 16.dp, vertical = 8.dp)
    .background(Color.Blue, shape = RoundedCornerShape(8.dp))
    .clip(RoundedCornerShape(12.dp))
    .border(1.dp, Color.Gray, RoundedCornerShape(8.dp))
    .clickable { /* acción */ }
    .clickable(
        interactionSource = remember { MutableInteractionSource() },
        indication = null,                      // Sin ripple
    ) { }
    .semantics { contentDescription = "Botón de compartir" }  // Accesibilidad
    .testTag("share_button")                    // Para tests UI
    .weight(1f)                                 // Dentro de Row/Column
    .align(Alignment.CenterHorizontally)        // Dentro de Column
    .offset(x = 8.dp, y = (-4).dp)
    .rotate(45f)
    .scale(1.2f)
    .alpha(0.5f)
    .zIndex(1f)
```

---

## 🖼️ Widgets visuales comunes

```kotlin
// Texto
Text(
    text = "Hola mundo",
    style = MaterialTheme.typography.headlineMedium,
    color = MaterialTheme.colorScheme.onSurface,
    textAlign = TextAlign.Center,
    maxLines = 2,
    overflow = TextOverflow.Ellipsis,
    fontWeight = FontWeight.Bold,
)

// Texto con múltiples estilos
Text(
    text = buildAnnotatedString {
        append("Precio: ")
        withStyle(SpanStyle(fontWeight = FontWeight.Bold, color = Color.Green)) {
            append("\$99.99")
        }
    }
)

// Imagen con Coil
AsyncImage(
    model = ImageRequest.Builder(LocalContext.current)
        .data(product.imageUrl)
        .crossfade(true)
        .build(),
    contentDescription = product.name,
    modifier = Modifier.fillMaxWidth().height(200.dp),
    contentScale = ContentScale.Crop,
    placeholder = painterResource(R.drawable.placeholder),
    error = painterResource(R.drawable.error),
)

// Botones
Button(onClick = {}) { Text("Primario") }
OutlinedButton(onClick = {}) { Text("Outlined") }
TextButton(onClick = {}) { Text("Text") }
FilledTonalButton(onClick = {}) { Text("Tonal") }
ElevatedButton(onClick = {}) { Text("Elevated") }
IconButton(onClick = {}) { Icon(Icons.Default.Favorite, contentDescription = "Like") }
FloatingActionButton(onClick = {}) { Icon(Icons.Default.Add, contentDescription = "Add") }
ExtendedFloatingActionButton(
    onClick = {},
    icon = { Icon(Icons.Default.Add, contentDescription = null) },
    text = { Text("Nuevo") },
)

// TextField
var text by remember { mutableStateOf("") }
OutlinedTextField(
    value = text,
    onValueChange = { text = it },
    label = { Text("Email") },
    leadingIcon = { Icon(Icons.Default.Email, contentDescription = null) },
    trailingIcon = { if (text.isNotEmpty()) IconButton(onClick = { text = "" }) {
        Icon(Icons.Default.Clear, contentDescription = "Limpiar")
    }},
    keyboardOptions = KeyboardOptions(
        keyboardType = KeyboardType.Email,
        imeAction = ImeAction.Next,
    ),
    keyboardActions = KeyboardActions(onNext = { /* focus next */ }),
    singleLine = true,
    isError = text.isNotEmpty() && !text.contains("@"),
    supportingText = { if (text.isNotEmpty() && !text.contains("@")) Text("Email inválido") },
    modifier = Modifier.fillMaxWidth(),
)
```

---

## 🧭 Navigation 3 (nuevo sistema — recomendado)

> Navigation 3 es la nueva librería oficial de navegación para Compose, anunciada en 2025. A diferencia de Navigation 2, **vos sos dueño del back stack** — es una simple `List` de claves que podés manipular directamente. Actualmente en alpha.

### Dependencias

```kotlin
// build.gradle.kts
dependencies {
    val nav3Version = "1.0.0-alpha04"
    implementation("androidx.navigation3:navigation3-ui:$nav3Version")
    implementation("androidx.lifecycle:lifecycle-viewmodel-navigation3:2.9.0-alpha04")

    // Para rememberNavBackStack con persistencia
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.7.3")
}
```

### Conceptos clave

| Concepto | Descripción |
|---|---|
| **Back stack** | `SnapshotStateList<T>` de claves que representan el historial. Vos lo gestionás. |
| **Key** | Cualquier tipo (generalmente `data class` o `data object`) que identifica un destino. |
| **NavEntry** | Envuelve una clave + su contenido Composable + metadata opcional. |
| **entryProvider** | Función que convierte una clave en su `NavEntry` correspondiente. |
| **NavDisplay** | Composable que observa el back stack y renderiza el destino activo. |

### Implementación básica

```kotlin
// 1. Definir claves de navegación
// Para back stack simple (sin persistencia)
data object HomeKey
data class ProductDetailKey(val id: String)
data class SearchKey(val query: String = "")

// Para back stack persistente (sobrevive proceso muerte / rotación)
@Serializable data object HomeKey : NavKey
@Serializable data class ProductDetailKey(val id: String) : NavKey
@Serializable data class SearchKey(val query: String = "") : NavKey

// 2. Crear el back stack
@Composable
fun MyApp() {
    // Sin persistencia — usar cuando no necesitás guardar estado de nav
    val backStack = remember { mutableStateListOf<Any>(HomeKey) }

    // Con persistencia (recomendado) — sobrevive rotación y muerte del proceso
    // Las claves DEBEN implementar NavKey y tener @Serializable
    val backStack = rememberNavBackStack(HomeKey)

    NavDisplay(
        backStack = backStack,
        onBack = { backStack.removeLastOrNull() },
        entryProvider = entryProvider {
            entry<HomeKey> {
                HomeScreen(
                    onProductClick = { id -> backStack.add(ProductDetailKey(id)) },
                    onSearchClick = { backStack.add(SearchKey()) },
                )
            }
            entry<ProductDetailKey> { key ->
                ProductDetailScreen(
                    productId = key.id,
                    onBack = { backStack.removeLastOrNull() },
                )
            }
            entry<SearchKey> { key ->
                SearchScreen(
                    initialQuery = key.query,
                    onBack = { backStack.removeLastOrNull() },
                )
            }
        },
    )
}
```

### entryProvider — dos formas de usarlo

```kotlin
// Forma 1: DSL (recomendada) — más limpio, maneja el fallback automáticamente
entryProvider = entryProvider {
    entry<HomeKey> { HomeScreen() }
    entry<ProductDetailKey> { key -> ProductDetailScreen(id = key.id) }
    entry<SearchKey>(
        metadata = mapOf("fullScreen" to true)    // metadata opcional para el layout
    ) { SearchScreen() }
}

// Forma 2: lambda directa con when
entryProvider = { key ->
    when (key) {
        is HomeKey -> NavEntry(key) { HomeScreen() }
        is ProductDetailKey -> NavEntry(key) { ProductDetailScreen(id = key.id) }
        else -> NavEntry(Unit) { Text("Destino desconocido") }
    }
}
```

### Operaciones sobre el back stack

```kotlin
// Navegar hacia adelante (push)
backStack.add(ProductDetailKey("abc-123"))

// Volver (pop)
backStack.removeLastOrNull()

// Reemplazar el destino actual
backStack[backStack.lastIndex] = ProductDetailKey("nuevo-id")

// Volver hasta un destino específico (inclusive)
backStack.removeAll { it is ProductDetailKey }

// Limpiar y navegar a raíz
backStack.clear()
backStack.add(HomeKey)

// Verificar si hay historial para volver
val canGoBack = backStack.size > 1
```

### ViewModel con scope por NavEntry

```kotlin
// Scopear ViewModels a cada entrada del back stack
// (se crea al entrar, se destruye al salir del back stack)

// build.gradle.kts
implementation("androidx.lifecycle:lifecycle-viewmodel-navigation3:2.9.0-alpha04")

// En NavDisplay — agregar los decoradores necesarios
NavDisplay(
    backStack = backStack,
    onBack = { backStack.removeLastOrNull() },
    entryDecorators = listOf(
        rememberSaveableStateHolderNavEntryDecorator(),   // Guarda estado Composable
        rememberViewModelStoreNavEntryDecorator(),         // Scopea ViewModels a NavEntry
    ),
    entryProvider = entryProvider {
        entry<ProductDetailKey> { key ->
            // Este ViewModel se destruye cuando ProductDetailKey sale del back stack
            val viewModel: ProductDetailViewModel = viewModel()
            ProductDetailScreen(viewModel = viewModel)
        }
    },
)
```

### BottomBar con Navigation 3

```kotlin
@Composable
fun MainScreen() {
    val backStack = rememberNavBackStack(HomeTab)
    val currentKey = backStack.lastOrNull()

    Scaffold(
        bottomBar = {
            NavigationBar {
                NavigationBarItem(
                    selected = currentKey is HomeTab,
                    onClick = {
                        // Limpiar hasta la raíz y establecer tab seleccionado
                        backStack.clear()
                        backStack.add(HomeTab)
                    },
                    icon = { Icon(Icons.Default.Home, contentDescription = null) },
                    label = { Text("Inicio") },
                )
                NavigationBarItem(
                    selected = currentKey is ProfileTab,
                    onClick = {
                        backStack.clear()
                        backStack.add(ProfileTab)
                    },
                    icon = { Icon(Icons.Default.Person, contentDescription = null) },
                    label = { Text("Perfil") },
                )
            }
        }
    ) { padding ->
        NavDisplay(
            modifier = Modifier.padding(padding),
            backStack = backStack,
            onBack = { backStack.removeLastOrNull() },
            entryProvider = entryProvider {
                entry<HomeTab> { HomeScreen(onNavigate = { backStack.add(it) }) }
                entry<ProfileTab> { ProfileScreen() }
                entry<ProductDetailKey> { key -> ProductDetailScreen(id = key.id) }
            },
        )
    }
}
```

### Navegación condicional (auth guard)

```kotlin
@Composable
fun AppRoot() {
    val isLoggedIn by authViewModel.isLoggedIn.collectAsStateWithLifecycle()
    val backStack = rememberNavBackStack(if (isLoggedIn) HomeKey else LoginKey)

    // Redirigir si cambia el estado de auth
    LaunchedEffect(isLoggedIn) {
        if (isLoggedIn) {
            backStack.clear()
            backStack.add(HomeKey)
        } else {
            backStack.clear()
            backStack.add(LoginKey)
        }
    }

    NavDisplay(
        backStack = backStack,
        onBack = { backStack.removeLastOrNull() },
        entryProvider = entryProvider {
            entry<LoginKey> { LoginScreen(onSuccess = { backStack.add(HomeKey) }) }
            entry<HomeKey> { HomeScreen() }
        },
    )
}
```

---

## 🧭 Navigation 2 (legacy — referencia)

> Sigue funcionando y es estable. Úsalo si aún no migraste o preferís la API declarativa con grafo.

```kotlin
// Navigation 2 — type-safe con Kotlin Serialization (desde 2.8)
@Serializable object HomeRoute
@Serializable data class DetailRoute(val productId: Int)

@Composable
fun AppNavGraph(navController: NavHostController = rememberNavController()) {
    NavHost(navController = navController, startDestination = HomeRoute) {
        composable<HomeRoute> {
            HomeScreen(onProductClick = { id -> navController.navigate(DetailRoute(id)) })
        }
        composable<DetailRoute> { backStack ->
            val args = backStack.toRoute<DetailRoute>()
            DetailScreen(productId = args.productId, onBack = { navController.popBackStack() })
        }
    }
}

// Navegar
navController.navigate(DetailRoute(productId = 42))
navController.popBackStack()
navController.navigate(HomeRoute) { popUpTo<HomeRoute> { inclusive = true } }
```

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

---

## ⚙️ WorkManager

```kotlin
// Worker con Koin
class SyncWorker(
    context: Context,
    params: WorkerParameters,
    private val repository: ProductRepository,   // Inyectado por Koin
) : CoroutineWorker(context, params) {

    override suspend fun doWork(): Result {
        return try {
            val category = inputData.getString(KEY_CATEGORY) ?: ""
            repository.refreshProducts(category)

            // Output data para encadenar workers
            val outputData = workDataOf(KEY_COUNT to repository.getCount())
            Result.success(outputData)
        } catch (e: Exception) {
            if (runAttemptCount < 3) Result.retry() else Result.failure()
        }
    }

    companion object {
        const val KEY_CATEGORY = "category"
        const val KEY_COUNT = "count"
        const val WORK_NAME = "sync_products"
    }
}

// Módulo Koin para Workers
val workersModule = module {
    worker { SyncWorker(get(), get(), get()) }
}

// App.kt — registrar factory de Koin en WorkManager
startKoin {
    workManagerFactory()   // IMPORTANTE: conecta Koin con WorkManager
    modules(workersModule, /* ... */)
}

// Programar trabajo
class WorkScheduler(private val workManager: WorkManager) {

    // Una sola vez
    fun syncNow(category: String) {
        val inputData = workDataOf(SyncWorker.KEY_CATEGORY to category)
        val constraints = Constraints.Builder()
            .setRequiredNetworkType(NetworkType.CONNECTED)
            .setRequiresBatteryNotLow(true)
            .build()

        val request = OneTimeWorkRequestBuilder<SyncWorker>()
            .setInputData(inputData)
            .setConstraints(constraints)
            .setBackoffCriteria(BackoffPolicy.EXPONENTIAL, 30, TimeUnit.SECONDS)
            .addTag("sync")
            .build()

        workManager.enqueueUniqueWork(
            SyncWorker.WORK_NAME,
            ExistingWorkPolicy.REPLACE,
            request,
        )
    }

    // Periódico (mínimo 15 minutos)
    fun schedulePeriodicSync() {
        val request = PeriodicWorkRequestBuilder<SyncWorker>(1, TimeUnit.HOURS)
            .setConstraints(Constraints.Builder()
                .setRequiredNetworkType(NetworkType.CONNECTED)
                .build())
            .build()

        workManager.enqueueUniquePeriodicWork(
            "periodic_sync",
            ExistingPeriodicWorkPolicy.KEEP,
            request,
        )
    }

    // Encadenar workers
    fun chainedWork() {
        val sync = OneTimeWorkRequestBuilder<SyncWorker>().build()
        val cleanup = OneTimeWorkRequestBuilder<CleanupWorker>().build()
        val notify = OneTimeWorkRequestBuilder<NotifyWorker>().build()

        workManager
            .beginUniqueWork("chain", ExistingWorkPolicy.REPLACE, sync)
            .then(cleanup)
            .then(notify)
            .enqueue()
    }

    // Observar estado
    fun observeSync(lifecycleOwner: LifecycleOwner) {
        workManager.getWorkInfosForUniqueWorkLiveData(SyncWorker.WORK_NAME)
            .observe(lifecycleOwner) { workInfos ->
                workInfos.forEach { info ->
                    when (info.state) {
                        WorkInfo.State.SUCCEEDED -> { /* Completado */ }
                        WorkInfo.State.FAILED -> { /* Falló */ }
                        WorkInfo.State.RUNNING -> { /* En progreso */ }
                        else -> Unit
                    }
                }
            }
    }
}
```

---

## 📡 Broadcast Receivers

```kotlin
// BroadcastReceiver básico
class NetworkReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        when (intent.action) {
            ConnectivityManager.CONNECTIVITY_ACTION -> {
                val connected = isConnected(context)
                // Notificar a la app
            }
            Intent.ACTION_BOOT_COMPLETED -> {
                // Re-programar alarmas o workers
            }
        }
    }

    private fun isConnected(context: Context): Boolean {
        val cm = context.getSystemService(Context.CONNECTIVITY_SERVICE) as ConnectivityManager
        return cm.activeNetwork?.let {
            cm.getNetworkCapabilities(it)?.hasCapability(NetworkCapabilities.NET_CAPABILITY_INTERNET)
        } ?: false
    }
}

// Registrar en AndroidManifest.xml (para acciones del sistema al iniciar)
// <receiver android:name=".NetworkReceiver" android:exported="false">
//     <intent-filter>
//         <action android:name="android.intent.action.BOOT_COMPLETED"/>
//     </intent-filter>
// </receiver>

// Registrar dinámicamente (recomendado para la mayoría de casos)
class MainActivity : ComponentActivity() {
    private val networkReceiver = NetworkReceiver()

    override fun onStart() {
        super.onStart()
        val filter = IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION)
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            registerReceiver(networkReceiver, filter, RECEIVER_NOT_EXPORTED)
        } else {
            registerReceiver(networkReceiver, filter)
        }
    }

    override fun onStop() {
        super.onStop()
        unregisterReceiver(networkReceiver)
    }
}

// Receiver con Flow para consumir en ViewModel (moderno)
fun Context.networkStatusFlow(): Flow<Boolean> = callbackFlow {
    val receiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context, intent: Intent) {
            trySend(isNetworkAvailable(context))
        }
    }
    val filter = IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION)
    registerReceiver(receiver, filter)
    trySend(isNetworkAvailable(this@networkStatusFlow))  // Emitir estado inicial

    awaitClose { unregisterReceiver(receiver) }
}.distinctUntilChanged()

// Broadcast local (entre componentes de la misma app)
val localBroadcastManager = LocalBroadcastManager.getInstance(context)

// Enviar
val intent = Intent("com.app.ACTION_REFRESH").apply { putExtra("key", "value") }
localBroadcastManager.sendBroadcast(intent)

// Recibir
val receiver = object : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) { /* ... */ }
}
localBroadcastManager.registerReceiver(receiver, IntentFilter("com.app.ACTION_REFRESH"))
localBroadcastManager.unregisterReceiver(receiver)
```

---

## 🔔 Notificaciones

```kotlin
// NotificationHelper.kt
class NotificationHelper(private val context: Context) {

    companion object {
        const val CHANNEL_GENERAL = "channel_general"
        const val CHANNEL_SYNC = "channel_sync"
    }

    fun createChannels() {
        val channels = listOf(
            NotificationChannel(CHANNEL_GENERAL, "General", NotificationManager.IMPORTANCE_DEFAULT)
                .apply { description = "Notificaciones generales" },
            NotificationChannel(CHANNEL_SYNC, "Sincronización", NotificationManager.IMPORTANCE_LOW)
                .apply { description = "Estado de sincronización" },
        )
        val manager = context.getSystemService(NotificationManager::class.java)
        channels.forEach { manager.createNotificationChannel(it) }
    }

    fun showNotification(title: String, body: String, id: Int = 1) {
        val intent = Intent(context, MainActivity::class.java).apply {
            flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK
        }
        val pendingIntent = PendingIntent.getActivity(
            context, 0, intent,
            PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE,
        )

        val notification = NotificationCompat.Builder(context, CHANNEL_GENERAL)
            .setSmallIcon(R.drawable.ic_notification)
            .setContentTitle(title)
            .setContentText(body)
            .setStyle(NotificationCompat.BigTextStyle().bigText(body))
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)
            .setContentIntent(pendingIntent)
            .setAutoCancel(true)
            .build()

        NotificationManagerCompat.from(context).notify(id, notification)
    }
}

// App.kt — crear canales al iniciar
class App : Application() {
    override fun onCreate() {
        super.onCreate()
        NotificationHelper(this).createChannels()
        // ...
    }
}
```

---

## 🎨 Temas con Material 3

```kotlin
// ui/theme/Color.kt
val PrimaryBlue = Color(0xFF2563EB)
val OnPrimaryBlue = Color(0xFFFFFFFF)
val SurfaceLight = Color(0xFFF8FAFC)
val SurfaceDark = Color(0xFF1E293B)

// ui/theme/Theme.kt
private val LightColorScheme = lightColorScheme(
    primary = PrimaryBlue,
    onPrimary = OnPrimaryBlue,
    surface = SurfaceLight,
)

private val DarkColorScheme = darkColorScheme(
    primary = Color(0xFF93C5FD),
    surface = SurfaceDark,
)

@Composable
fun MyAppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = true,     // Material You (Android 12+)
    content: @Composable () -> Unit,
) {
    val colorScheme = when {
        dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            val context = LocalContext.current
            if (darkTheme) dynamicDarkColorScheme(context) else dynamicLightColorScheme(context)
        }
        darkTheme -> DarkColorScheme
        else -> LightColorScheme
    }

    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography,
        content = content,
    )
}

// Acceder al tema en composables
val colors = MaterialTheme.colorScheme
val typography = MaterialTheme.typography
val shapes = MaterialTheme.shapes

Text("Hola", style = typography.headlineMedium, color = colors.primary)
Surface(color = colors.surfaceVariant, shape = shapes.medium) { /* ... */ }
```

---

## 🎬 Animaciones

```kotlin
// AnimatedVisibility
AnimatedVisibility(
    visible = isVisible,
    enter = fadeIn() + slideInVertically(),
    exit = fadeOut() + slideOutVertically(),
) {
    Text("Aparezco animado")
}

// animateContentSize — anima cambios de tamaño
Box(modifier = Modifier.animateContentSize(animationSpec = spring())) {
    Text(if (expanded) longText else shortText)
}

// animate*AsState — animar un valor
val alpha by animateFloatAsState(
    targetValue = if (isVisible) 1f else 0f,
    animationSpec = tween(300),
    label = "alpha",
)
val color by animateColorAsState(
    targetValue = if (isSelected) MaterialTheme.colorScheme.primary else Color.Gray,
    label = "color",
)

// Crossfade entre composables
Crossfade(targetState = currentScreen, label = "screen_transition") { screen ->
    when (screen) {
        Screen.Home -> HomeContent()
        Screen.Detail -> DetailContent()
    }
}

// updateTransition — animaciones coordinadas
val transition = updateTransition(targetState = isExpanded, label = "expand")
val height by transition.animateDp(label = "height") { expanded -> if (expanded) 200.dp else 56.dp }
val alpha by transition.animateFloat(label = "alpha") { expanded -> if (expanded) 1f else 0f }

// Animación con rememberInfiniteTransition
val infiniteTransition = rememberInfiniteTransition(label = "pulse")
val scale by infiniteTransition.animateFloat(
    initialValue = 1f,
    targetValue = 1.1f,
    animationSpec = infiniteRepeatable(
        animation = tween(1000),
        repeatMode = RepeatMode.Reverse,
    ),
    label = "scale",
)
```

---

## 🧪 Testing

```kotlin
// Unit test — ViewModel
class ProductsViewModelTest {
    private val getProductsUseCase: GetProductsUseCase = mockk()
    private lateinit var viewModel: ProductsViewModel

    @Before
    fun setup() {
        viewModel = ProductsViewModel(getProductsUseCase)
    }

    @Test
    fun `loadProducts emite Success cuando el use case retorna datos`() = runTest {
        val products = listOf(Product(id = 1, name = "Test"))
        coEvery { getProductsUseCase() } returns Result.success(products)

        viewModel.loadProducts()

        val state = viewModel.uiState.value
        assertIs<UiState.Success>(state)
        assertEquals(products, state.products)
    }
}

// Composable UI test
class ProductsScreenTest {
    @get:Rule
    val composeTestRule = createComposeRule()

    @Test
    fun `muestra lista cuando el estado es Success`() {
        val products = listOf(Product(id = 1, name = "Laptop"))

        composeTestRule.setContent {
            MyAppTheme {
                ProductsScreen(
                    uiState = UiState.Success(products),
                    onDelete = {},
                )
            }
        }

        composeTestRule.onNodeWithText("Laptop").assertIsDisplayed()
        composeTestRule.onNodeWithTag("product_list").assertIsDisplayed()
    }

    @Test
    fun `muestra loading indicator cuando carga`() {
        composeTestRule.setContent {
            MyAppTheme { ProductsScreen(uiState = UiState.Loading, onDelete = {}) }
        }
        composeTestRule.onNodeWithContentDescription("Cargando").assertIsDisplayed()
    }
}
```

---

## 📦 Paquetes y librerías esenciales

| Categoría | Librería | Descripción |
|---|---|---|
| DI | `koin-android` + `koin-androidx-compose` | Inyección de dependencias liviana |
| HTTP | `retrofit` + `okhttp` | Cliente HTTP + interceptores |
| Serialización | `kotlinx-serialization-json` | JSON moderno de Kotlin |
| Base de datos | `room` | ORM local con soporte Flow |
| Preferencias | `datastore-preferences` | Alternativa moderna a SharedPreferences |
| Paginación | `paging3` + `paging-compose` | Listas paginadas con Room/API |
| Workers | `work-runtime-ktx` | Tareas en background garantizadas |
| Imágenes | `coil-compose` | Carga de imágenes async con cache |
| Navegación | `navigation-compose` | Navegación declarativa oficial |
| Coroutines | `kotlinx-coroutines-android` | Concurrencia asíncrona |
| Logging | `timber` | Logging con tags automáticos |
| Mocking | `mockk` | Mocking idiomático para Kotlin |
| Tests | `turbine` | Testing de Flows |
| Linting | `detekt` | Análisis estático avanzado |
| Inyección | `hilt` | DI de Google (alternativa a Koin) |

---

## 💡 Buenas prácticas

- **UDF (Unidirectional Data Flow)**: estado baja por parámetros, eventos suben con lambdas. El ViewModel nunca recibe Context ni referencias a Composables.
- **`collectAsStateWithLifecycle`** en lugar de `collectAsState` — respeta el ciclo de vida y cancela la colección cuando la app va a background.
- **`cachedIn(viewModelScope)`** en flows de Paging — imprescindible para no re-cargar datos al rotar la pantalla.
- **Hoisting del estado al nivel adecuado** — solo subir el estado hasta el ancestro común más bajo que lo necesite.
- **`remember` con `key`** — cuando el valor debe recalcularse al cambiar una dependencia: `remember(userId) { computeValue(userId) }`.
- **Usar `key()` en LazyColumn** — evita recomposiciones innecesarias y mantiene el estado de los ítems al reordenar.
- **`derivedStateOf`** — cuando un estado se deriva de otro y no querés recomposición en cada cambio: `val isButtonEnabled by remember { derivedStateOf { text.length > 3 } }`.
- **Workers para trabajo crítico** — WorkManager garantiza ejecución aunque la app se cierre o el dispositivo reinicie.
- **Separar la lógica de Room en el hilo IO** — Room con suspend y Flow maneja esto automáticamente, no llamar desde el hilo principal.
- **Migrations explícitas en Room** — nunca depender de `fallbackToDestructiveMigration` en producción.
- **Testear ViewModels sin Android** — usar `runTest` de `kotlinx-coroutines-test` y `Turbine` para flujos.

---

## 📋 AndroidManifest.xml — Permisos completos

### Estructura básica del Manifest

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <!-- Permisos que necesita la app -->
    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:name=".App"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/Theme.MyApp"
        android:usesCleartextTraffic="false">

        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:windowSoftInputMode="adjustResize">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

    </application>
</manifest>
```

---

### Tipos de permisos según su nivel de protección

| Nivel | Descripción | Se necesita pedir en runtime? |
|---|---|---|
| `normal` | Riesgo bajo, se otorga automáticamente al instalar | No |
| `dangerous` | Accede a datos sensibles del usuario | **Sí** (Android 6+) |
| `signature` | Solo apps firmadas con el mismo certificado | No |
| `signatureOrSystem` | Apps del sistema o misma firma | No |

---

### 🌐 Red y conectividad

```xml
<!-- Acceso a Internet — normal -->
<uses-permission android:name="android.permission.INTERNET" />

<!-- Ver estado de la red (WiFi/datos) — normal -->
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

<!-- Ver estado del WiFi — normal -->
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />

<!-- Cambiar conectividad WiFi — normal -->
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />

<!-- Cambiar conectividad de red — normal -->
<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />

<!-- Bluetooth (descubrir/conectar) — dangerous (Android 12+) -->
<uses-permission android:name="android.permission.BLUETOOTH_SCAN"
    android:usesPermissionFlags="neverForLocation" />
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
<uses-permission android:name="android.permission.BLUETOOTH_ADVERTISE" />

<!-- NFC — normal -->
<uses-permission android:name="android.permission.NFC" />
```

---

### 📍 Ubicación

```xml
<!-- Ubicación aproximada (ciudad/barrio) — dangerous -->
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />

<!-- Ubicación precisa (GPS) — dangerous -->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />

<!-- Ubicación en background (siempre, incluso con app cerrada) — dangerous -->
<!-- REQUIERE pedir ACCESS_FINE/COARSE primero, luego pedir este por separado -->
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
```

> ⚠️ En Android 10+ para ubicación en background el usuario debe ir manualmente a Ajustes y seleccionar "Permitir siempre".

---

### 📷 Cámara y multimedia

```xml
<!-- Cámara — dangerous -->
<uses-permission android:name="android.permission.CAMERA" />

<!-- Micrófono (grabar audio) — dangerous -->
<uses-permission android:name="android.permission.RECORD_AUDIO" />

<!-- Leer imágenes (Android 13+) — dangerous -->
<uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />

<!-- Leer videos (Android 13+) — dangerous -->
<uses-permission android:name="android.permission.READ_MEDIA_VIDEO" />

<!-- Leer audio (Android 13+) — dangerous -->
<uses-permission android:name="android.permission.READ_MEDIA_AUDIO" />

<!-- Leer almacenamiento externo (hasta Android 12) — dangerous -->
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"
    android:maxSdkVersion="32" />

<!-- Escribir en almacenamiento externo (hasta Android 9) — dangerous -->
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
    android:maxSdkVersion="28" />

<!-- Selección de fotos parcial (Android 14+) — dangerous -->
<uses-permission android:name="android.permission.READ_MEDIA_VISUAL_USER_SELECTED" />
```

---

### 👤 Contactos y calendarios

```xml
<!-- Leer contactos — dangerous -->
<uses-permission android:name="android.permission.READ_CONTACTS" />

<!-- Escribir/modificar contactos — dangerous -->
<uses-permission android:name="android.permission.WRITE_CONTACTS" />

<!-- Obtener lista de cuentas del dispositivo — dangerous -->
<uses-permission android:name="android.permission.GET_ACCOUNTS" />

<!-- Leer eventos del calendario — dangerous -->
<uses-permission android:name="android.permission.READ_CALENDAR" />

<!-- Crear/modificar eventos del calendario — dangerous -->
<uses-permission android:name="android.permission.WRITE_CALENDAR" />
```

---

### 📞 Teléfono y SMS

```xml
<!-- Hacer llamadas — dangerous -->
<uses-permission android:name="android.permission.CALL_PHONE" />

<!-- Leer estado del teléfono (número, IMEI) — dangerous -->
<uses-permission android:name="android.permission.READ_PHONE_STATE" />

<!-- Leer números de teléfono — dangerous -->
<uses-permission android:name="android.permission.READ_PHONE_NUMBERS" />

<!-- Responder llamadas — dangerous -->
<uses-permission android:name="android.permission.ANSWER_PHONE_CALLS" />

<!-- Leer SMS — dangerous -->
<uses-permission android:name="android.permission.READ_SMS" />

<!-- Enviar SMS — dangerous -->
<uses-permission android:name="android.permission.SEND_SMS" />

<!-- Recibir SMS — dangerous -->
<uses-permission android:name="android.permission.RECEIVE_SMS" />

<!-- Leer log de llamadas — dangerous -->
<uses-permission android:name="android.permission.READ_CALL_LOG" />
```

---

### 🔔 Notificaciones y alarmas

```xml
<!-- Mostrar notificaciones (Android 13+) — dangerous -->
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />

<!-- Programar alarmas exactas (Android 12+) — special -->
<uses-permission android:name="android.permission.SCHEDULE_EXACT_ALARM" />

<!-- Alarmas exactas con permiso especial (Android 13+) — special -->
<uses-permission android:name="android.permission.USE_EXACT_ALARM" />
```

---

### 🔋 Batería y background

```xml
<!-- Ignorar optimización de batería (correr en background) — special -->
<!-- El usuario debe habilitarlo manualmente desde Ajustes -->
<uses-permission android:name="android.permission.REQUEST_IGNORE_BATTERY_OPTIMIZATIONS" />

<!-- Mantener CPU activa con WakeLock — normal -->
<uses-permission android:name="android.permission.WAKE_LOCK" />

<!-- Arrancar al iniciar el dispositivo — normal -->
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />

<!-- Servicio en primer plano — normal -->
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />

<!-- Tipo específico de foreground service (Android 14+) -->
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_CAMERA" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_LOCATION" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_MICROPHONE" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_MEDIA_PLAYBACK" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_DATA_SYNC" />
```

---

### 🏃 Sensores y actividad física

```xml
<!-- Reconocimiento de actividad física (pasos, etc.) — dangerous (Android 10+) -->
<uses-permission android:name="android.permission.ACTIVITY_RECOGNITION" />

<!-- Sensor de frecuencia cardíaca (Wear OS) — dangerous -->
<uses-permission android:name="android.permission.BODY_SENSORS" />

<!-- Sensor de frecuencia en background (Wear OS) — dangerous -->
<uses-permission android:name="android.permission.BODY_SENSORS_BACKGROUND" />
```

---

### 🔒 Permisos especiales (requieren Intent a Ajustes)

Estos permisos no se piden con `requestPermissions()` sino redirigiendo al usuario a la pantalla de ajustes:

```xml
<!-- Instalar apps de fuentes desconocidas -->
<uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES" />

<!-- Dibujar sobre otras apps (overlay) -->
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />

<!-- Acceso a configuración del sistema -->
<uses-permission android:name="android.permission.WRITE_SETTINGS" />

<!-- Acceso a notificaciones de otras apps -->
<uses-permission android:name="android.permission.BIND_NOTIFICATION_LISTENER_SERVICE" />

<!-- Gestor de contraseñas / AutoFill -->
<uses-permission android:name="android.permission.BIND_AUTOFILL_SERVICE" />
```

```kotlin
// Verificar y redirigir a ajustes para permisos especiales
if (!Settings.canDrawOverlays(context)) {
    val intent = Intent(
        Settings.ACTION_MANAGE_OVERLAY_PERMISSION,
        Uri.parse("package:${context.packageName}")
    )
    startActivity(intent)
}

if (!Environment.isExternalStorageManager()) {
    val intent = Intent(Settings.ACTION_MANAGE_APP_ALL_FILES_ACCESS_PERMISSION)
    startActivity(intent)
}
```

---

### 📲 Pedir permisos en runtime (Compose)

```kotlin
// Permiso único
val launcher = rememberLauncherForActivityResult(
    ActivityResultContracts.RequestPermission()
) { isGranted ->
    if (isGranted) { /* usar la funcionalidad */ }
    else { /* mostrar rationale o deshabilitar feature */ }
}

Button(onClick = { launcher.launch(Manifest.permission.CAMERA) }) {
    Text("Abrir cámara")
}

// Múltiples permisos a la vez
val multiplePermissionsLauncher = rememberLauncherForActivityResult(
    ActivityResultContracts.RequestMultiplePermissions()
) { permissions ->
    val cameraGranted = permissions[Manifest.permission.CAMERA] ?: false
    val micGranted = permissions[Manifest.permission.RECORD_AUDIO] ?: false
    if (cameraGranted && micGranted) { /* iniciar videollamada */ }
}

Button(onClick = {
    multiplePermissionsLauncher.launch(
        arrayOf(Manifest.permission.CAMERA, Manifest.permission.RECORD_AUDIO)
    )
}) {
    Text("Iniciar videollamada")
}

// Verificar si ya tiene el permiso
val hasPermission = ContextCompat.checkSelfPermission(
    context,
    Manifest.permission.CAMERA
) == PackageManager.PERMISSION_GRANTED

// Verificar si mostrar rationale (usuario denegó previamente sin "no preguntar más")
val shouldShowRationale = ActivityCompat.shouldShowRequestPermissionRationale(
    activity,
    Manifest.permission.CAMERA
)
```

---

### 📦 `uses-feature` — declarar hardware requerido

```xml
<!-- La app requiere cámara (excluye del Play Store dispositivos sin cámara) -->
<uses-feature android:name="android.hardware.camera" android:required="true" />

<!-- Cámara trasera -->
<uses-feature android:name="android.hardware.camera.any" android:required="false" />

<!-- GPS -->
<uses-feature android:name="android.hardware.location.gps" android:required="false" />

<!-- Bluetooth -->
<uses-feature android:name="android.hardware.bluetooth" android:required="false" />

<!-- Pantalla táctil -->
<uses-feature android:name="android.hardware.touchscreen" android:required="false" />

<!-- NFC -->
<uses-feature android:name="android.hardware.nfc" android:required="false" />

<!-- Sensor de huellas -->
<uses-feature android:name="android.hardware.fingerprint" android:required="false" />
```

> Usar `android:required="false"` para features opcionales — así la app está disponible en más dispositivos y vos verificás en código si el hardware existe.

---

## 📨 Intents — Comunicación entre componentes

### Tipos de Intent

| Tipo | Descripción | Ejemplo |
|---|---|---|
| **Explícito** | Especifica el componente exacto (clase) a iniciar | Abrir tu propia Activity |
| **Implícito** | Declara una acción; el sistema elige qué app la maneja | Compartir texto, abrir URL |
| **Broadcast** | Mensaje de difusión a múltiples receptores | Batería baja, boot completado |
| **Pending Intent** | Intent diferido que otra app puede ejecutar en tu nombre | Notificaciones, alarmas, widgets |

---

### Intent explícito

```kotlin
// Abrir otra Activity de tu propia app
val intent = Intent(context, DetailActivity::class.java).apply {
    putExtra("product_id", 42)
    putExtra("product_name", "Laptop")
    putExtra("price", 999.99)
}
startActivity(intent)

// Leer extras en la Activity destino
val productId = intent.getIntExtra("product_id", -1)
val name = intent.getStringExtra("product_name") ?: ""
val price = intent.getDoubleExtra("price", 0.0)

// Pasar objetos Parcelable
@Parcelize
data class Product(val id: Int, val name: String) : Parcelable

val intent = Intent(context, DetailActivity::class.java).apply {
    putExtra("product", product)
}

val product = intent.getParcelableExtra<Product>("product")          // hasta API 32
val product = intent.getParcelableExtra("product", Product::class.java) // API 33+

// Iniciar Service explícito
val serviceIntent = Intent(context, SyncService::class.java).apply {
    action = "com.app.ACTION_SYNC"
    putExtra("force", true)
}
context.startService(serviceIntent)
context.startForegroundService(serviceIntent) // Para foreground services

// Iniciar con resultado (Activity Result API — moderno)
val launcher = rememberLauncherForActivityResult(
    ActivityResultContracts.StartActivityForResult()
) { result ->
    if (result.resultCode == Activity.RESULT_OK) {
        val data = result.data?.getStringExtra("result_key")
    }
}
launcher.launch(Intent(context, PickerActivity::class.java))

// Devolver resultado desde la Activity destino
setResult(Activity.RESULT_OK, Intent().apply { putExtra("result_key", "valor") })
finish()
```

---

### Intent implícito — acciones del sistema

```kotlin
// Abrir URL en el navegador
val intent = Intent(Intent.ACTION_VIEW, Uri.parse("https://developer.android.com"))
startActivity(intent)

// Abrir teléfono con número marcado (no llama — muestra el dialer)
val intent = Intent(Intent.ACTION_DIAL, Uri.parse("tel:+595981123456"))
startActivity(intent)

// Llamar directamente (requiere CALL_PHONE)
val intent = Intent(Intent.ACTION_CALL, Uri.parse("tel:+595981123456"))
startActivity(intent)

// Enviar email
val intent = Intent(Intent.ACTION_SENDTO).apply {
    data = Uri.parse("mailto:destino@email.com")
    putExtra(Intent.EXTRA_SUBJECT, "Asunto del email")
    putExtra(Intent.EXTRA_TEXT, "Cuerpo del mensaje")
}
startActivity(intent)

// Compartir texto (chooser — muestra selector de apps)
val intent = Intent(Intent.ACTION_SEND).apply {
    type = "text/plain"
    putExtra(Intent.EXTRA_TEXT, "Texto a compartir")
    putExtra(Intent.EXTRA_TITLE, "Título del share")
}
startActivity(Intent.createChooser(intent, "Compartir via..."))

// Compartir imagen
val intent = Intent(Intent.ACTION_SEND).apply {
    type = "image/jpeg"
    putExtra(Intent.EXTRA_STREAM, imageUri)
    addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION) // IMPORTANTE para URIs de FileProvider
}
startActivity(Intent.createChooser(intent, "Compartir imagen"))

// Compartir múltiples archivos
val intent = Intent(Intent.ACTION_SEND_MULTIPLE).apply {
    type = "image/*"
    putParcelableArrayListExtra(Intent.EXTRA_STREAM, ArrayList(uriList))
    addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION)
}
startActivity(Intent.createChooser(intent, "Compartir imágenes"))

// Abrir ajustes de la app
val intent = Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS).apply {
    data = Uri.parse("package:${context.packageName}")
}
startActivity(intent)

// Abrir ajustes de ubicación
startActivity(Intent(Settings.ACTION_LOCATION_SOURCE_SETTINGS))

// Abrir Google Maps con ubicación
val uri = Uri.parse("geo:${lat},${lng}?q=${lat},${lng}(Nombre del lugar)")
startActivity(Intent(Intent.ACTION_VIEW, uri))

// Abrir Google Maps con ruta
val uri = Uri.parse("https://maps.google.com/maps?daddr=${destLat},${destLng}")
startActivity(Intent(Intent.ACTION_VIEW, uri))

// Verificar si hay una app para manejar el Intent ANTES de lanzarlo
val packageManager = context.packageManager
if (intent.resolveActivity(packageManager) != null) {
    startActivity(intent)
} else {
    // No hay app disponible — mostrar mensaje al usuario
}
```

---

### Intent Flags importantes

```kotlin
val intent = Intent(context, HomeActivity::class.java).apply {

    // Limpiar el back stack y crear nueva tarea — útil para login/logout
    flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK

    // No crear nueva instancia si ya está en el back stack
    addFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP)

    // Llevar al frente si ya existe en alguna tarea
    addFlags(Intent.FLAG_ACTIVITY_REORDER_TO_FRONT)

    // Limpiar actividades encima hasta encontrar esta en el back stack
    addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP)

    // Otorgar permiso de lectura a la URI incluida (FileProvider)
    addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION)

    // Otorgar permiso de escritura a la URI incluida
    addFlags(Intent.FLAG_GRANT_WRITE_URI_PERMISSION)
}
```

---

### PendingIntent — para notificaciones, alarmas y widgets

```kotlin
// PendingIntent para abrir Activity desde una notificación
val intent = Intent(context, MainActivity::class.java).apply {
    flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK
    putExtra("notification_id", 123)
}
val pendingIntent = PendingIntent.getActivity(
    context,
    requestCode = 0,
    intent,
    PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE, // FLAG_IMMUTABLE requerido en Android 12+
)

// PendingIntent para Broadcast (botón en notificación)
val broadcastIntent = Intent(context, NotificationActionReceiver::class.java).apply {
    action = "com.app.ACTION_DISMISS"
    putExtra("notification_id", 123)
}
val broadcastPending = PendingIntent.getBroadcast(
    context,
    requestCode = 1,
    broadcastIntent,
    PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE,
)

// Usar en notificación
NotificationCompat.Builder(context, CHANNEL_ID)
    .setContentIntent(pendingIntent)          // Toque en la notificación
    .addAction(R.drawable.ic_close, "Cerrar", broadcastPending)  // Botón
    .build()

// PendingIntent para AlarmManager
val alarmManager = context.getSystemService(AlarmManager::class.java)
val triggerTime = System.currentTimeMillis() + TimeUnit.MINUTES.toMillis(30)

alarmManager.setExactAndAllowWhileIdle(
    AlarmManager.RTC_WAKEUP,
    triggerTime,
    pendingIntent,
)
```

---

### Deep Links — recibir Intents en tu app

```xml
<!-- AndroidManifest.xml — configurar intent-filter para deep links -->
<activity android:name=".MainActivity" android:exported="true">
    <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <!-- https://miapp.com/product/123 -->
        <data android:scheme="https" android:host="miapp.com" android:pathPrefix="/product" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <!-- miapp://product/123 -->
        <data android:scheme="miapp" android:host="product" />
    </intent-filter>
</activity>
```

```kotlin
// Leer el deep link en la Activity
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    handleIntent(intent)
}

override fun onNewIntent(intent: Intent) {
    super.onNewIntent(intent)
    handleIntent(intent)
}

private fun handleIntent(intent: Intent) {
    if (intent.action == Intent.ACTION_VIEW) {
        val uri = intent.data ?: return
        // https://miapp.com/product/123
        val productId = uri.lastPathSegment   // "123"
        val source = uri.getQueryParameter("source") // ?source=email
    }
}
```

---

## 📂 Almacenamiento — leer y escribir en directorios públicos

### Resumen de opciones de almacenamiento

| Opción | Dónde | Accesible por otras apps | Se borra al desinstalar |
|---|---|---|---|
| Internal files | `/data/data/<pkg>/files/` | No | Sí |
| Internal cache | `/data/data/<pkg>/cache/` | No | Sí (+ limpiado por el SO) |
| External privado | `sdcard/Android/data/<pkg>/` | No (Android 10+) | Sí |
| **MediaStore** | `sdcard/Pictures/`, `Music/`, etc. | **Sí** | No (persiste) |
| **SAF (Storage Access Framework)** | Cualquier directorio | **Con permiso del usuario** | No |
| Downloads público | `sdcard/Download/` | Sí | No |

---

### MediaStore — escribir en directorios públicos (imágenes, video, audio)

MediaStore es el sistema recomendado para guardar archivos en directorios públicos como `Pictures`, `Movies`, `Music`, `Downloads` sin necesitar permisos de almacenamiento en Android 10+.

```kotlin
// Guardar imagen en Pictures/NombreApp/
suspend fun saveImageToGallery(context: Context, bitmap: Bitmap, filename: String): Uri? {
    val contentValues = ContentValues().apply {
        put(MediaStore.Images.Media.DISPLAY_NAME, "$filename.jpg")
        put(MediaStore.Images.Media.MIME_TYPE, "image/jpeg")
        put(MediaStore.Images.Media.RELATIVE_PATH, "Pictures/MiApp") // Subcarpeta dentro de Pictures
        put(MediaStore.Images.Media.IS_PENDING, 1)  // Marcar como pendiente durante escritura
    }

    val resolver = context.contentResolver
    val uri = resolver.insert(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, contentValues)
        ?: return null

    return try {
        resolver.openOutputStream(uri)?.use { stream ->
            bitmap.compress(Bitmap.CompressFormat.JPEG, 90, stream)
        }
        // Marcar como completo — lo hace visible en la galería
        contentValues.clear()
        contentValues.put(MediaStore.Images.Media.IS_PENDING, 0)
        resolver.update(uri, contentValues, null, null)
        uri
    } catch (e: Exception) {
        resolver.delete(uri, null, null) // Limpiar si falla
        null
    }
}

// Guardar video en Movies/NombreApp/
suspend fun saveVideoToMovies(context: Context, sourceFile: File, filename: String): Uri? {
    val contentValues = ContentValues().apply {
        put(MediaStore.Video.Media.DISPLAY_NAME, "$filename.mp4")
        put(MediaStore.Video.Media.MIME_TYPE, "video/mp4")
        put(MediaStore.Video.Media.RELATIVE_PATH, "Movies/MiApp")
        put(MediaStore.Video.Media.IS_PENDING, 1)
    }

    val resolver = context.contentResolver
    val uri = resolver.insert(MediaStore.Video.Media.EXTERNAL_CONTENT_URI, contentValues)
        ?: return null

    return try {
        resolver.openOutputStream(uri)?.use { output ->
            sourceFile.inputStream().use { input -> input.copyTo(output) }
        }
        contentValues.clear()
        contentValues.put(MediaStore.Video.Media.IS_PENDING, 0)
        resolver.update(uri, contentValues, null, null)
        uri
    } catch (e: Exception) {
        resolver.delete(uri, null, null)
        null
    }
}

// Guardar archivo en Downloads/
suspend fun saveToDownloads(context: Context, content: String, filename: String): Uri? {
    val contentValues = ContentValues().apply {
        put(MediaStore.Downloads.DISPLAY_NAME, filename)           // Ej: "reporte.csv"
        put(MediaStore.Downloads.MIME_TYPE, "text/csv")
        put(MediaStore.Downloads.RELATIVE_PATH, "Download/MiApp") // Subcarpeta en Downloads
        put(MediaStore.Downloads.IS_PENDING, 1)
    }

    val resolver = context.contentResolver
    val uri = resolver.insert(MediaStore.Downloads.EXTERNAL_CONTENT_URI, contentValues)
        ?: return null

    return try {
        resolver.openOutputStream(uri)?.use { stream ->
            stream.write(content.toByteArray(Charsets.UTF_8))
        }
        contentValues.clear()
        contentValues.put(MediaStore.Downloads.IS_PENDING, 0)
        resolver.update(uri, contentValues, null, null)
        uri
    } catch (e: Exception) {
        resolver.delete(uri, null, null)
        null
    }
}
```

---

### MediaStore — leer archivos públicos

```kotlin
// Leer imágenes de la galería con permisos (Android 13+: READ_MEDIA_IMAGES)
fun queryImages(context: Context): List<MediaImage> {
    val images = mutableListOf<MediaImage>()
    val projection = arrayOf(
        MediaStore.Images.Media._ID,
        MediaStore.Images.Media.DISPLAY_NAME,
        MediaStore.Images.Media.DATE_ADDED,
        MediaStore.Images.Media.SIZE,
        MediaStore.Images.Media.MIME_TYPE,
    )
    val sortOrder = "${MediaStore.Images.Media.DATE_ADDED} DESC"

    context.contentResolver.query(
        MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
        projection,
        null,   // selection (WHERE)
        null,   // selectionArgs
        sortOrder,
    )?.use { cursor ->
        val idColumn = cursor.getColumnIndexOrThrow(MediaStore.Images.Media._ID)
        val nameColumn = cursor.getColumnIndexOrThrow(MediaStore.Images.Media.DISPLAY_NAME)
        val dateColumn = cursor.getColumnIndexOrThrow(MediaStore.Images.Media.DATE_ADDED)

        while (cursor.moveToNext()) {
            val id = cursor.getLong(idColumn)
            val name = cursor.getString(nameColumn)
            val date = cursor.getLong(dateColumn)
            val uri = ContentUris.withAppendedId(
                MediaStore.Images.Media.EXTERNAL_CONTENT_URI, id
            )
            images.add(MediaImage(uri = uri, name = name, dateAdded = date))
        }
    }
    return images
}

// Filtrar por directorio o tipo
context.contentResolver.query(
    MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
    projection,
    selection = "${MediaStore.Images.Media.RELATIVE_PATH} LIKE ?",
    selectionArgs = arrayOf("Pictures/MiApp/%"),
    sortOrder = null,
)

// Leer contenido de un archivo por URI
fun readTextFromUri(context: Context, uri: Uri): String? {
    return context.contentResolver.openInputStream(uri)?.use { stream ->
        stream.bufferedReader().readText()
    }
}

// Obtener bitmap de una URI (Coil lo hace automáticamente, pero para uso manual)
fun loadBitmapFromUri(context: Context, uri: Uri): Bitmap? {
    return if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
        val source = ImageDecoder.createSource(context.contentResolver, uri)
        ImageDecoder.decodeBitmap(source)
    } else {
        @Suppress("DEPRECATION")
        MediaStore.Images.Media.getBitmap(context.contentResolver, uri)
    }
}
```

---

### SAF (Storage Access Framework) — acceso a cualquier directorio

El SAF permite al usuario elegir archivos o carpetas desde cualquier proveedor (almacenamiento local, Google Drive, etc.) sin que la app necesite permisos de almacenamiento.

```kotlin
// Abrir un archivo para LEER (el usuario elige)
val openFileLauncher = rememberLauncherForActivityResult(
    ActivityResultContracts.OpenDocument()
) { uri: Uri? ->
    uri ?: return@rememberLauncherForActivityResult
    // URI persistente — pedir permiso de persistencia
    context.contentResolver.takePersistableUriPermission(
        uri,
        Intent.FLAG_GRANT_READ_URI_PERMISSION,
    )
    val content = context.contentResolver.openInputStream(uri)?.use {
        it.bufferedReader().readText()
    }
}
// Abrir solo PDFs y documentos
openFileLauncher.launch(arrayOf("application/pdf", "text/plain", "text/csv"))

// Abrir múltiples archivos
val openMultipleLauncher = rememberLauncherForActivityResult(
    ActivityResultContracts.OpenMultipleDocuments()
) { uris: List<Uri> ->
    uris.forEach { uri -> /* procesar cada archivo */ }
}
openMultipleLauncher.launch(arrayOf("image/*"))

// Crear un archivo nuevo (el usuario elige dónde guardar)
val createFileLauncher = rememberLauncherForActivityResult(
    ActivityResultContracts.CreateDocument("text/plain")
) { uri: Uri? ->
    uri ?: return@rememberLauncherForActivityResult
    context.contentResolver.openOutputStream(uri)?.use { stream ->
        stream.write("Contenido del archivo".toByteArray())
    }
}
createFileLauncher.launch("mi_archivo.txt")  // Nombre sugerido

// Elegir directorio completo (acceso persistente a una carpeta)
val openDirLauncher = rememberLauncherForActivityResult(
    ActivityResultContracts.OpenDocumentTree()
) { treeUri: Uri? ->
    treeUri ?: return@rememberLauncherForActivityResult
    // Pedir permisos persistentes (sobrevive reinicios)
    context.contentResolver.takePersistableUriPermission(
        treeUri,
        Intent.FLAG_GRANT_READ_URI_PERMISSION or Intent.FLAG_GRANT_WRITE_URI_PERMISSION,
    )
    // Listar archivos del directorio
    val docTree = DocumentFile.fromTreeUri(context, treeUri)
    docTree?.listFiles()?.forEach { file ->
        Log.d("SAF", "Archivo: ${file.name}, tipo: ${file.type}")
    }
}
openDirLauncher.launch(null) // null = directorio raíz por defecto

// Leer/escribir en un directorio con permisos persistentes (sesiones futuras)
fun writeToPersistedDir(context: Context, treeUri: Uri, filename: String, content: String) {
    val docTree = DocumentFile.fromTreeUri(context, treeUri) ?: return
    val newFile = docTree.createFile("text/plain", filename) ?: return
    context.contentResolver.openOutputStream(newFile.uri)?.use { stream ->
        stream.write(content.toByteArray())
    }
}

// Verificar permisos persistentes ya otorgados
fun getPersistedDirectories(context: Context): List<Uri> {
    return context.contentResolver.persistedUriPermissions
        .filter { it.isReadPermission && it.isWritePermission }
        .map { it.uri }
}
```

---

### FileProvider — compartir archivos internos con otras apps

FileProvider permite compartir archivos del almacenamiento privado de tu app (internal/external) con otras apps de forma segura, sin exponer rutas del filesystem.

```xml
<!-- AndroidManifest.xml -->
<provider
    android:name="androidx.core.content.FileProvider"
    android:authorities="${applicationId}.fileprovider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/file_paths" />
</provider>
```

```xml
<!-- res/xml/file_paths.xml -->
<?xml version="1.0" encoding="utf-8"?>
<paths>
    <!-- Almacenamiento interno: context.filesDir -->
    <files-path name="internal_files" path="." />

    <!-- Cache interno: context.cacheDir -->
    <cache-path name="internal_cache" path="." />

    <!-- Almacenamiento externo privado: context.getExternalFilesDir(null) -->
    <external-files-path name="external_files" path="." />

    <!-- Cache externo: context.externalCacheDir -->
    <external-cache-path name="external_cache" path="." />

    <!-- Directorio específico dentro de files -->
    <files-path name="shared_images" path="images/" />
</paths>
```

```kotlin
// Obtener URI segura para compartir un archivo interno
fun getShareableUri(context: Context, file: File): Uri {
    return FileProvider.getUriForFile(
        context,
        "${context.packageName}.fileprovider",
        file,
    )
}

// Compartir un PDF generado internamente
val pdfFile = File(context.filesDir, "reporte.pdf")
// ... generar el PDF ...
val uri = getShareableUri(context, pdfFile)

val shareIntent = Intent(Intent.ACTION_SEND).apply {
    type = "application/pdf"
    putExtra(Intent.EXTRA_STREAM, uri)
    addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION)  // OBLIGATORIO para FileProvider
}
startActivity(Intent.createChooser(shareIntent, "Compartir PDF"))

// Abrir archivo con otra app (visor de PDF, editor, etc.)
val viewIntent = Intent(Intent.ACTION_VIEW).apply {
    setDataAndType(uri, "application/pdf")
    addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION)
}
startActivity(viewIntent)

// Capturar foto con la cámara y guardar en archivo interno
val photoFile = File(context.cacheDir, "temp_photo.jpg")
val photoUri = getShareableUri(context, photoFile)

val cameraLauncher = rememberLauncherForActivityResult(
    ActivityResultContracts.TakePicture()
) { success ->
    if (success) {
        // La foto fue guardada en photoFile
        val bitmap = BitmapFactory.decodeFile(photoFile.absolutePath)
    }
}
cameraLauncher.launch(photoUri)
```

---

### Almacenamiento interno — archivos privados de la app

```kotlin
// Escribir en internal storage
fun writeInternalFile(context: Context, filename: String, content: String) {
    context.openFileOutput(filename, Context.MODE_PRIVATE).use { stream ->
        stream.write(content.toByteArray())
    }
}

// Leer desde internal storage
fun readInternalFile(context: Context, filename: String): String? {
    return try {
        context.openFileInput(filename).use { stream ->
            stream.bufferedReader().readText()
        }
    } catch (e: FileNotFoundException) { null }
}

// Listar archivos internos
val files = context.fileList()  // Array<String> de nombres

// Eliminar archivo interno
context.deleteFile("mi_archivo.txt")

// Acceso por File para operaciones más complejas
val file = File(context.filesDir, "subdirectorio/archivo.json")
file.parentFile?.mkdirs()  // Crear subdirectorios si no existen
file.writeText(jsonString)
val content = file.readText()

// Cache interno (el SO puede limpiar este directorio)
val cacheFile = File(context.cacheDir, "thumbnail_cache/img_123.jpg")
cacheFile.parentFile?.mkdirs()
```

---

### Almacenamiento externo privado (sin permisos en Android 10+)

```kotlin
// Directorio específico en sdcard/Android/data/<package>/files/
val picturesDir = context.getExternalFilesDir(Environment.DIRECTORY_PICTURES)
val downloadsDir = context.getExternalFilesDir(Environment.DIRECTORY_DOWNLOADS)
val documentsDir = context.getExternalFilesDir(Environment.DIRECTORY_DOCUMENTS)
val musicDir = context.getExternalFilesDir(Environment.DIRECTORY_MUSIC)
val moviesDir = context.getExternalFilesDir(Environment.DIRECTORY_MOVIES)

// Guardar imagen en directorio externo privado
val imageFile = File(picturesDir, "foto_perfil.jpg")
bitmap.compress(Bitmap.CompressFormat.JPEG, 90, imageFile.outputStream())

// Verificar disponibilidad del almacenamiento externo
val isAvailable = Environment.getExternalStorageState() == Environment.MEDIA_MOUNTED
val isReadOnly = Environment.getExternalStorageState() == Environment.MEDIA_MOUNTED_READ_ONLY

// Verificar espacio disponible
val statFs = StatFs(context.getExternalFilesDir(null)?.path ?: "")
val availableBytes = statFs.availableBlocksLong * statFs.blockSizeLong
val totalBytes = statFs.blockCountLong * statFs.blockSizeLong
```

---

### Acceso completo al almacenamiento (MANAGE_EXTERNAL_STORAGE)

Para casos excepcionales como administradores de archivos. Requiere justificación en Google Play.

```xml
<uses-permission android:name="android.permission.MANAGE_EXTERNAL_STORAGE"
    tools:ignore="ScopedStorage" />
```

```kotlin
// Verificar y solicitar
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R) {
    if (!Environment.isExternalStorageManager()) {
        val intent = Intent(Settings.ACTION_MANAGE_APP_ALL_FILES_ACCESS_PERMISSION).apply {
            data = Uri.parse("package:${context.packageName}")
        }
        startActivity(intent)
    } else {
        // Tiene acceso completo — puede usar File() con rutas absolutas
        val dcimDir = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DCIM)
        val files = dcimDir.listFiles()
    }
}
```

---

### MIME types comunes

| Tipo | MIME type |
|---|---|
| Imagen JPEG | `image/jpeg` |
| Imagen PNG | `image/png` |
| Imagen GIF | `image/gif` |
| Imagen WebP | `image/webp` |
| Cualquier imagen | `image/*` |
| Video MP4 | `video/mp4` |
| Cualquier video | `video/*` |
| Audio MP3 | `audio/mpeg` |
| Cualquier audio | `audio/*` |
| PDF | `application/pdf` |
| JSON | `application/json` |
| ZIP | `application/zip` |
| Texto plano | `text/plain` |
| CSV | `text/csv` |
| HTML | `text/html` |
| Cualquier archivo | `*/*` |

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
    .throttleFirst(1_000)                    // Emitir máximo una vez por segundo
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

---

## 🔥 Firebase

### Setup

```kotlin
// build.gradle.kts (project)
plugins {
    id("com.google.gms.google-services") version "4.4.2" apply false
    id("com.google.firebase.crashlytics") version "3.0.2" apply false
}

// build.gradle.kts (app)
plugins {
    id("com.google.gms.google-services")
    id("com.google.firebase.crashlytics")
}

dependencies {
    val firebaseBom = platform("com.google.firebase:firebase-bom:33.7.0")
    implementation(firebaseBom)
    implementation("com.google.firebase:firebase-auth-ktx")
    implementation("com.google.firebase:firebase-firestore-ktx")
    implementation("com.google.firebase:firebase-storage-ktx")
    implementation("com.google.firebase:firebase-crashlytics-ktx")
    implementation("com.google.firebase:firebase-analytics-ktx")
    implementation("com.google.firebase:firebase-config-ktx")
    implementation("com.google.firebase:firebase-messaging-ktx")

    // Google Sign-In
    implementation("com.google.android.gms:play-services-auth:21.3.0")
    // Credential Manager (nuevo — reemplaza GoogleSignInClient)
    implementation("androidx.credentials:credentials:1.5.0")
    implementation("androidx.credentials:credentials-play-services-auth:1.5.0")
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

---

## 🔒 Seguridad

### EncryptedSharedPreferences y Keystore

```kotlin
// EncryptedSharedPreferences — almacenamiento seguro de clave-valor
fun createEncryptedPrefs(context: Context): SharedPreferences {
    val mainKey = MasterKey.Builder(context)
        .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
        .build()

    return EncryptedSharedPreferences.create(
        context,
        "secure_prefs",
        mainKey,
        EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
        EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM,
    )
}

// Uso igual que SharedPreferences normal
val prefs = createEncryptedPrefs(context)
prefs.edit {
    putString("auth_token", token)
    putString("refresh_token", refreshToken)
}
val token = prefs.getString("auth_token", null)

// Dependencias necesarias
// implementation("androidx.security:security-crypto:1.1.0-alpha06")
```

---

### BiometricPrompt — huella digital y reconocimiento facial

```kotlin
// build.gradle.kts
// implementation("androidx.biometric:biometric:1.2.0-alpha05")

class BiometricAuthManager(private val activity: FragmentActivity) {

    fun isBiometricAvailable(): Boolean {
        val biometricManager = BiometricManager.from(activity)
        return biometricManager.canAuthenticate(
            BiometricManager.Authenticators.BIOMETRIC_STRONG or
            BiometricManager.Authenticators.DEVICE_CREDENTIAL
        ) == BiometricManager.BIOMETRIC_SUCCESS
    }

    fun authenticate(
        title: String = "Autenticación requerida",
        subtitle: String = "Usá tu huella o PIN para continuar",
        onSuccess: () -> Unit,
        onError: (String) -> Unit,
        onFailed: () -> Unit = {},
    ) {
        val executor = ContextCompat.getMainExecutor(activity)
        val callback = object : BiometricPrompt.AuthenticationCallback() {
            override fun onAuthenticationSucceeded(result: BiometricPrompt.AuthenticationResult) {
                onSuccess()
            }
            override fun onAuthenticationError(errorCode: Int, errString: CharSequence) {
                if (errorCode != BiometricPrompt.ERROR_USER_CANCELED &&
                    errorCode != BiometricPrompt.ERROR_NEGATIVE_BUTTON) {
                    onError(errString.toString())
                }
            }
            override fun onAuthenticationFailed() = onFailed()
        }

        val promptInfo = BiometricPrompt.PromptInfo.Builder()
            .setTitle(title)
            .setSubtitle(subtitle)
            .setAllowedAuthenticators(
                BiometricManager.Authenticators.BIOMETRIC_STRONG or
                BiometricManager.Authenticators.DEVICE_CREDENTIAL
            )
            .build()

        BiometricPrompt(activity, executor, callback).authenticate(promptInfo)
    }
}

// Uso en Composable
@Composable
fun BiometricLoginButton() {
    val activity = LocalContext.current as FragmentActivity
    val biometricManager = remember { BiometricAuthManager(activity) }

    if (biometricManager.isBiometricAvailable()) {
        Button(onClick = {
            biometricManager.authenticate(
                onSuccess = { /* proceder */ },
                onError = { message -> /* mostrar error */ },
            )
        }) {
            Icon(Icons.Default.Fingerprint, contentDescription = null)
            Spacer(Modifier.width(8.dp))
            Text("Ingresar con huella")
        }
    }
}
```

---

### Network Security Config — HTTPS y certificate pinning

```xml
<!-- res/xml/network_security_config.xml -->
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <!-- Permitir tráfico HTTP solo en debug hacia emulador -->
    <domain-config cleartextTrafficPermitted="false">
        <domain includeSubdomains="true">api.miapp.com</domain>
        <pin-set expiration="2026-12-31">
            <pin digest="SHA-256">AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=</pin>
            <pin digest="SHA-256">BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=</pin>
        </pin-set>
    </domain-config>
    <debug-overrides>
        <trust-anchors>
            <certificates src="user" />  <!-- Confiar en certs del usuario en debug (Charles/Proxyman) -->
        </trust-anchors>
    </debug-overrides>
</network-security-config>
```

```xml
<!-- AndroidManifest.xml -->
<application
    android:networkSecurityConfig="@xml/network_security_config"
    android:usesCleartextTraffic="false">
```

---

## 🔧 Foreground Service

```kotlin
// MusicPlayerService.kt
class MusicPlayerService : Service() {

    private val binder = LocalBinder()
    inner class LocalBinder : Binder() {
        fun getService(): MusicPlayerService = this@MusicPlayerService
    }

    override fun onBind(intent: Intent): IBinder = binder

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        when (intent?.action) {
            ACTION_PLAY -> play(intent.getStringExtra("track_url"))
            ACTION_PAUSE -> pause()
            ACTION_STOP -> stopSelf()
        }
        return START_NOT_STICKY  // No recrear si el sistema lo mata
        // START_STICKY          → recrear con intent null
        // START_REDELIVER_INTENT → recrear con último intent
    }

    private fun play(url: String?) {
        val notification = buildNotification()
        // Android 14+ requiere declarar el tipo de foreground service
        ServiceCompat.startForeground(
            this,
            NOTIFICATION_ID,
            notification,
            ServiceInfo.FOREGROUND_SERVICE_TYPE_MEDIA_PLAYBACK,
        )
        // ... lógica de reproducción ...
    }

    private fun pause() { /* pausar media player */ }

    private fun buildNotification(): Notification {
        val stopPending = PendingIntent.getService(
            this, 0,
            Intent(this, MusicPlayerService::class.java).apply { action = ACTION_STOP },
            PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE,
        )
        return NotificationCompat.Builder(this, CHANNEL_ID)
            .setSmallIcon(R.drawable.ic_music)
            .setContentTitle("Reproduciendo")
            .setContentText("Nombre de la canción")
            .addAction(R.drawable.ic_stop, "Detener", stopPending)
            .setOngoing(true)        // No puede ser descartada con swipe
            .setSilent(true)
            .build()
    }

    override fun onDestroy() {
        super.onDestroy()
        ServiceCompat.stopForeground(this, ServiceCompat.STOP_FOREGROUND_REMOVE)
    }

    companion object {
        const val ACTION_PLAY = "com.app.PLAY"
        const val ACTION_PAUSE = "com.app.PAUSE"
        const val ACTION_STOP = "com.app.STOP"
        const val NOTIFICATION_ID = 101
        const val CHANNEL_ID = "media_playback"
    }
}
```

```xml
<!-- AndroidManifest.xml -->
<service
    android:name=".MusicPlayerService"
    android:exported="false"
    android:foregroundServiceType="mediaPlayback" />
```

```kotlin
// Iniciar el service desde Composable/Activity
val context = LocalContext.current

// Iniciar
Intent(context, MusicPlayerService::class.java).also { intent ->
    intent.action = MusicPlayerService.ACTION_PLAY
    intent.putExtra("track_url", "https://...")
    context.startForegroundService(intent)
}

// Bindear para comunicación bidireccional
var service: MusicPlayerService? = null
val connection = object : ServiceConnection {
    override fun onServiceConnected(name: ComponentName, binder: IBinder) {
        service = (binder as MusicPlayerService.LocalBinder).getService()
    }
    override fun onServiceDisconnected(name: ComponentName) { service = null }
}
context.bindService(
    Intent(context, MusicPlayerService::class.java),
    connection,
    Context.BIND_AUTO_CREATE,
)
// No olvidar unbindService(connection) en onStop/onDestroy
```

---

## 🔄 Ciclo de vida y estado persistente

### rememberSaveable y SavedStateHandle

```kotlin
// rememberSaveable — sobrevive rotación Y muerte del proceso (si es serializable)
var text by rememberSaveable { mutableStateOf("") }
var count by rememberSaveable { mutableIntStateOf(0) }

// Para objetos complejos — implementar Saver
data class SearchState(val query: String, val filters: List<String>)

val SearchStateSaver = Saver<SearchState, Bundle>(
    save = { state ->
        Bundle().apply {
            putString("query", state.query)
            putStringArrayList("filters", ArrayList(state.filters))
        }
    },
    restore = { bundle ->
        SearchState(
            query = bundle.getString("query", ""),
            filters = bundle.getStringArrayList("filters") ?: emptyList(),
        )
    }
)

var searchState by rememberSaveable(stateSaver = SearchStateSaver) {
    mutableStateOf(SearchState("", emptyList()))
}

// SavedStateHandle en ViewModel — sobrevive muerte del proceso
class SearchViewModel(
    savedStateHandle: SavedStateHandle,
) : ViewModel() {

    // Leer parámetro de navegación type-safe (Navigation 3 / Navigation 2)
    private val productId: String = checkNotNull(savedStateHandle["productId"])

    // Estado que sobrevive muerte del proceso
    var query by savedStateHandle.saveable { mutableStateOf("") }

    // Como StateFlow
    val query: StateFlow<String> = savedStateHandle.getStateFlow("query", "")
    fun onQueryChange(q: String) { savedStateHandle["query"] = q }
}

// BackHandler — interceptar el botón atrás en Compose
@Composable
fun EditScreen(hasUnsavedChanges: Boolean, onConfirmLeave: () -> Unit) {
    var showDialog by remember { mutableStateOf(false) }

    BackHandler(enabled = hasUnsavedChanges) {
        showDialog = true    // Interceptar solo si hay cambios sin guardar
    }

    if (showDialog) {
        AlertDialog(
            title = { Text("¿Descartar cambios?") },
            text = { Text("Tenés cambios sin guardar.") },
            confirmButton = {
                TextButton(onClick = { showDialog = false; onConfirmLeave() }) {
                    Text("Descartar")
                }
            },
            dismissButton = {
                TextButton(onClick = { showDialog = false }) { Text("Cancelar") }
            },
            onDismissRequest = { showDialog = false },
        )
    }
}
```

---

## 🎛️ Compose — widgets avanzados

### PullToRefresh

```kotlin
// implementation("androidx.compose.material3:material3:1.3.x") — incluido en M3

@Composable
fun ProductsScreen(viewModel: ProductsViewModel = koinViewModel()) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    val isRefreshing = uiState is UiState.Loading

    val pullToRefreshState = rememberPullToRefreshState()

    PullToRefreshBox(
        isRefreshing = isRefreshing,
        onRefresh = viewModel::refresh,
        state = pullToRefreshState,
    ) {
        LazyColumn(modifier = Modifier.fillMaxSize()) {
            // contenido
        }
    }
}
```

---

### SwipeToDismiss

```kotlin
@Composable
fun SwipeableProductCard(
    product: Product,
    onDelete: (Product) -> Unit,
) {
    val dismissState = rememberSwipeToDismissBoxState(
        confirmValueChange = { value ->
            if (value == SwipeToDismissBoxValue.EndToStart) {
                onDelete(product)
                true
            } else false
        }
    )

    SwipeToDismissBox(
        state = dismissState,
        backgroundContent = {
            val color by animateColorAsState(
                targetValue = if (dismissState.dismissDirection == SwipeToDismissBoxValue.EndToStart)
                    MaterialTheme.colorScheme.errorContainer else Color.Transparent,
                label = "swipe_bg",
            )
            Box(
                modifier = Modifier.fillMaxSize().background(color).padding(end = 16.dp),
                contentAlignment = Alignment.CenterEnd,
            ) {
                Icon(Icons.Default.Delete, contentDescription = "Eliminar",
                    tint = MaterialTheme.colorScheme.onErrorContainer)
            }
        },
        enableDismissFromStartToEnd = false,
    ) {
        ProductCard(product = product)
    }
}
```

---

### ModalBottomSheet, DatePicker y SearchBar

```kotlin
// ModalBottomSheet
var showSheet by remember { mutableStateOf(false) }

if (showSheet) {
    ModalBottomSheet(
        onDismissRequest = { showSheet = false },
        sheetState = rememberModalBottomSheetState(skipPartiallyExpanded = true),
    ) {
        Column(modifier = Modifier.padding(16.dp).navigationBarsPadding()) {
            Text("Opciones", style = MaterialTheme.typography.titleLarge)
            Spacer(Modifier.height(16.dp))
            // contenido del sheet
        }
    }
}

// DatePicker (Material 3)
var showDatePicker by remember { mutableStateOf(false) }
val datePickerState = rememberDatePickerState(
    initialSelectedDateMillis = System.currentTimeMillis()
)

if (showDatePicker) {
    DatePickerDialog(
        onDismissRequest = { showDatePicker = false },
        confirmButton = {
            TextButton(onClick = {
                showDatePicker = false
                val selectedMillis = datePickerState.selectedDateMillis
                // usar selectedMillis
            }) { Text("Confirmar") }
        },
        dismissButton = {
            TextButton(onClick = { showDatePicker = false }) { Text("Cancelar") }
        },
    ) {
        DatePicker(state = datePickerState)
    }
}

// TimePicker (Material 3)
val timePickerState = rememberTimePickerState(initialHour = 12, initialMinute = 0)
TimePicker(state = timePickerState)
// Leer: timePickerState.hour, timePickerState.minute

// SearchBar (Material 3)
var query by remember { mutableStateOf("") }
var active by remember { mutableStateOf(false) }

SearchBar(
    inputField = {
        SearchBarDefaults.InputField(
            query = query,
            onQueryChange = { query = it },
            onSearch = { active = false; viewModel.search(query) },
            expanded = active,
            onExpandedChange = { active = it },
            placeholder = { Text("Buscar productos...") },
            leadingIcon = { Icon(Icons.Default.Search, contentDescription = null) },
            trailingIcon = {
                if (query.isNotEmpty()) {
                    IconButton(onClick = { query = "" }) {
                        Icon(Icons.Default.Clear, contentDescription = "Limpiar")
                    }
                }
            },
        )
    },
    expanded = active,
    onExpandedChange = { active = it },
    modifier = Modifier.fillMaxWidth(),
) {
    // Sugerencias de búsqueda cuando está expandido
    LazyColumn {
        items(suggestions) { suggestion ->
            ListItem(
                headlineContent = { Text(suggestion) },
                modifier = Modifier.clickable { query = suggestion; active = false }
            )
        }
    }
}

// ExposedDropdownMenu
var expanded by remember { mutableStateOf(false) }
var selectedOption by remember { mutableStateOf(options.first()) }

ExposedDropdownMenuBox(
    expanded = expanded,
    onExpandedChange = { expanded = it },
) {
    OutlinedTextField(
        value = selectedOption,
        onValueChange = {},
        readOnly = true,
        label = { Text("Categoría") },
        trailingIcon = { ExposedDropdownMenuDefaults.TrailingIcon(expanded) },
        modifier = Modifier.menuAnchor().fillMaxWidth(),
    )
    ExposedDropdownMenu(
        expanded = expanded,
        onDismissRequest = { expanded = false },
    ) {
        options.forEach { option ->
            DropdownMenuItem(
                text = { Text(option) },
                onClick = { selectedOption = option; expanded = false },
            )
        }
    }
}

// LazyColumn con stickyHeader
LazyColumn {
    groupedItems.forEach { (header, items) ->
        stickyHeader {
            Surface(
                modifier = Modifier.fillMaxWidth(),
                color = MaterialTheme.colorScheme.surfaceVariant,
            ) {
                Text(
                    text = header,
                    modifier = Modifier.padding(horizontal = 16.dp, vertical = 8.dp),
                    style = MaterialTheme.typography.labelLarge,
                )
            }
        }
        items(items = items, key = { it.id }) { item ->
            ItemCard(item = item)
        }
    }
}
```

---

## ⚡ Compose — performance y recomposición

```kotlin
// derivedStateOf — recalcular solo cuando cambia la fuente
val lazyListState = rememberLazyListState()
val showFab by remember {
    derivedStateOf { lazyListState.firstVisibleItemIndex > 0 }
}

// Stable e Immutable — marcar clases para evitar recomposiciones
@Stable    // Notifica a Compose cuando cambia (manualmente)
class UserState(name: String) {
    var name by mutableStateOf(name)
}

@Immutable  // Compose confía en que nunca cambia internamente
data class ProductUiModel(val id: Int, val name: String, val price: Double)

// ImmutableList para listas estables (kotlinx.collections.immutable)
// implementation("org.jetbrains.kotlinx:kotlinx-collections-immutable:0.3.8")
@Immutable
data class ProductsUiState(val products: ImmutableList<ProductUiModel>)

// CompositionLocal — pasar datos implícitamente por el árbol
val LocalUserSession = compositionLocalOf<UserSession> {
    error("UserSession no proveída")
}
val LocalAppConfig = staticCompositionLocalOf<AppConfig> {
    error("AppConfig no proveída")  // staticCompositionLocalOf para valores que no cambian
}

// Proveer valores
CompositionLocalProvider(
    LocalUserSession provides userSession,
    LocalAppConfig provides appConfig,
) {
    AppContent()
}

// Consumir en cualquier hijo del árbol
@Composable
fun ProfileSection() {
    val session = LocalUserSession.current
    Text("Hola, ${session.displayName}")
}

// key() — forzar recreación de un composable
items.forEach { item ->
    key(item.id) {          // Compose recrea este subtree si item.id cambia
        ItemCard(item = item)
    }
}

// Evitar lambdas inestables con remember
// ❌ Crea una nueva lambda en cada recomposición
LazyColumn {
    items(products) { product ->
        ProductCard(onClick = { viewModel.select(product) })
    }
}

// ✅ Lambda estabilizada
LazyColumn {
    items(products, key = { it.id }) { product ->
        val onClick = remember(product.id) { { viewModel.select(product) } }
        ProductCard(onClick = onClick)
    }
}
```

---

## ♿ Accesibilidad

```kotlin
// semantics — describir el propósito de un elemento para TalkBack
Box(
    modifier = Modifier
        .semantics {
            contentDescription = "Producto: Laptop, precio: $999"
            role = Role.Button
            onClick(label = "Ver detalles") { true }
        }
        .clickable { }
)

// mergeDescendants — combinar múltiples elementos en uno para TalkBack
Row(modifier = Modifier.semantics(mergeDescendants = true) { }) {
    Icon(Icons.Default.Star, contentDescription = null)  // null → TalkBack lo ignora
    Text("4.5 de 5 estrellas")
}

// CustomAccessibilityAction — acciones adicionales en el menú de TalkBack
Card(
    modifier = Modifier.semantics {
        customActions = listOf(
            CustomAccessibilityAction("Agregar al carrito") {
                onAddToCart()
                true
            },
            CustomAccessibilityAction("Ver detalles") {
                onViewDetails()
                true
            },
        )
    }
) { ProductContent() }

// stateDescription — describir estado actual
Checkbox(
    checked = isChecked,
    onCheckedChange = { isChecked = it },
    modifier = Modifier.semantics {
        stateDescription = if (isChecked) "Seleccionado" else "No seleccionado"
    }
)

// heading — marcar como encabezado
Text(
    "Categorías",
    modifier = Modifier.semantics { heading() },
    style = MaterialTheme.typography.headlineMedium,
)

// LiveRegion — anunciar cambios automáticamente
Text(
    text = statusMessage,
    modifier = Modifier.semantics {
        liveRegion = LiveRegionMode.Polite   // Anuncia cambios sin interrumpir
    }
)

// focusable — hacer enfocable un elemento no interactivo
Box(modifier = Modifier.focusable())

// clearAndSetSemantics — reemplazar completamente la semántica de hijos
Box(
    modifier = Modifier.clearAndSetSemantics {
        contentDescription = "Rating: 4 de 5 estrellas"
    }
) {
    // Cinco íconos de estrella — TalkBack solo leerá el contentDescription de arriba
    repeat(5) { StarIcon() }
}
```

---

## 🌍 Localización e internacionalización

```xml
<!-- res/values/strings.xml (español por defecto) -->
<resources>
    <string name="app_name">Mi App</string>
    <string name="welcome">Bienvenido, %1$s</string>

    <!-- Plurales -->
    <plurals name="product_count">
        <item quantity="zero">Sin productos</item>
        <item quantity="one">%d producto</item>
        <item quantity="other">%d productos</item>
    </plurals>

    <!-- Plurales con formato -->
    <plurals name="items_in_cart">
        <item quantity="one">Tenés %d ítem en el carrito</item>
        <item quantity="other">Tenés %d ítems en el carrito</item>
    </plurals>
</resources>

<!-- res/values-en/strings.xml (inglés) -->
<resources>
    <string name="welcome">Welcome, %1$s</string>
    <plurals name="product_count">
        <item quantity="one">%d product</item>
        <item quantity="other">%d products</item>
    </plurals>
</resources>
```

```kotlin
// Usar strings en Compose
Text(text = stringResource(R.string.app_name))
Text(text = stringResource(R.string.welcome, userName))

// Plurales
Text(text = pluralStringResource(R.plurals.product_count, count, count))

// Formateo de números y monedas respetando locale
val formattedPrice = NumberFormat.getCurrencyInstance(Locale.getDefault()).format(price)
val formattedNumber = NumberFormat.getNumberInstance().format(bigNumber)

// Formateo de fechas con locale
val dateFormatter = DateTimeFormatter.ofLocalizedDate(FormatStyle.MEDIUM)
    .withLocale(Locale.getDefault())
val formatted = localDate.format(dateFormatter)

// Cambiar idioma en runtime (Android 13+ — per-app language)
AppCompatDelegate.setApplicationLocales(
    LocaleListCompat.forLanguageTags("es-AR")
)

// Detectar idioma actual
val currentLocale = AppCompatDelegate.getApplicationLocales()[0]
    ?: LocaleListCompat.getDefault()[0]

// Direccionalidad RTL/LTR
val layoutDirection = LocalLayoutDirection.current
if (layoutDirection == LayoutDirection.Rtl) { /* ajustar layout */ }
```

---

## 📱 App Widget con Glance

```kotlin
// build.gradle.kts
// implementation("androidx.glance:glance-appwidget:1.1.1")
// implementation("androidx.glance:glance-material3:1.1.1")

// Widget básico con Glance (Compose-like API)
class StatsWidget : GlanceAppWidget() {
    override suspend fun provideGlance(context: Context, id: GlanceId) {
        val stats = StatsRepository(context).getStats()   // cargar datos

        provideContent {
            GlanceTheme {
                StatsWidgetContent(stats = stats)
            }
        }
    }
}

@Composable
fun StatsWidgetContent(stats: Stats) {
    Column(
        modifier = GlanceModifier
            .fillMaxSize()
            .background(GlanceTheme.colors.surface)
            .padding(16.dp),
        verticalAlignment = Alignment.Vertical.CenterVertically,
    ) {
        Text(
            text = stats.title,
            style = TextStyle(
                fontWeight = FontWeight.Bold,
                fontSize = 16.sp,
                color = GlanceTheme.colors.onSurface,
            ),
        )
        Spacer(modifier = GlanceModifier.height(8.dp))
        Text(
            text = stats.value,
            style = TextStyle(fontSize = 32.sp, color = GlanceTheme.colors.primary),
        )
        // Botón que actualiza el widget
        Button(
            text = "Actualizar",
            onClick = actionRunCallback<RefreshWidgetAction>(),
        )
        // Botón que abre la app
        Button(
            text = "Abrir app",
            onClick = actionStartActivity<MainActivity>(),
        )
    }
}

// Acción del widget
class RefreshWidgetAction : ActionCallback {
    override suspend fun onAction(context: Context, glanceId: GlanceId, parameters: ActionParameters) {
        StatsWidget().update(context, glanceId)
    }
}

// Receiver — registro del widget
class StatsWidgetReceiver : GlanceAppWidgetReceiver() {
    override val glanceAppWidget: GlanceAppWidget = StatsWidget()
}
```

```xml
<!-- AndroidManifest.xml -->
<receiver
    android:name=".widget.StatsWidgetReceiver"
    android:exported="true">
    <intent-filter>
        <action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
    </intent-filter>
    <meta-data
        android:name="android.appwidget.provider"
        android:resource="@xml/stats_widget_info" />
</receiver>

<!-- res/xml/stats_widget_info.xml -->
```
```xml
<appwidget-provider
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:minWidth="180dp"
    android:minHeight="110dp"
    android:targetCellWidth="2"
    android:targetCellHeight="2"
    android:updatePeriodMillis="1800000"
    android:resizeMode="horizontal|vertical"
    android:widgetCategory="home_screen"
    android:initialLayout="@layout/glance_default_loading_layout" />
```

---

## 📶 Connectivity — monitoreo de red

```kotlin
// NetworkMonitor.kt — observar conectividad como Flow
class NetworkMonitor(private val context: Context) {

    val isOnline: Flow<Boolean> = callbackFlow {
        val connectivityManager = context.getSystemService(ConnectivityManager::class.java)

        // Emitir estado inicial
        trySend(connectivityManager.isCurrentlyConnected())

        val callback = object : ConnectivityManager.NetworkCallback() {
            override fun onAvailable(network: Network) { trySend(true) }
            override fun onLost(network: Network) { trySend(false) }
            override fun onCapabilitiesChanged(network: Network, caps: NetworkCapabilities) {
                trySend(caps.hasCapability(NetworkCapabilities.NET_CAPABILITY_INTERNET) &&
                        caps.hasCapability(NetworkCapabilities.NET_CAPABILITY_VALIDATED))
            }
        }

        val request = NetworkRequest.Builder()
            .addCapability(NetworkCapabilities.NET_CAPABILITY_INTERNET)
            .addTransportType(NetworkCapabilities.TRANSPORT_WIFI)
            .addTransportType(NetworkCapabilities.TRANSPORT_CELLULAR)
            .build()

        connectivityManager.registerNetworkCallback(request, callback)
        awaitClose { connectivityManager.unregisterNetworkCallback(callback) }
    }.distinctUntilChanged()

    private fun ConnectivityManager.isCurrentlyConnected(): Boolean {
        val network = activeNetwork ?: return false
        val caps = getNetworkCapabilities(network) ?: return false
        return caps.hasCapability(NetworkCapabilities.NET_CAPABILITY_INTERNET) &&
               caps.hasCapability(NetworkCapabilities.NET_CAPABILITY_VALIDATED)
    }
}

// Módulo Koin
val networkModule = module {
    single { NetworkMonitor(get()) }
}

// ViewModel con retry automático al volver online
class ProductsViewModel(
    private val repo: ProductRepository,
    private val networkMonitor: NetworkMonitor,
) : ViewModel() {

    init {
        // Recargar datos cuando vuelve la conexión
        viewModelScope.launch {
            networkMonitor.isOnline
                .filter { it }                        // Solo cuando vuelve a estar online
                .distinctUntilChanged()
                .drop(1)                              // Ignorar el estado inicial
                .collect { loadProducts() }
        }
    }
}

// Banner de sin conexión en Composable
@Composable
fun OfflineBanner() {
    val networkMonitor: NetworkMonitor = koinInject()
    val isOnline by networkMonitor.isOnline.collectAsStateWithLifecycle(initialValue = true)

    AnimatedVisibility(
        visible = !isOnline,
        enter = slideInVertically() + fadeIn(),
        exit = slideOutVertically() + fadeOut(),
    ) {
        Surface(color = MaterialTheme.colorScheme.errorContainer) {
            Row(
                modifier = Modifier.fillMaxWidth().padding(8.dp),
                horizontalArrangement = Arrangement.Center,
                verticalAlignment = Alignment.CenterVertically,
            ) {
                Icon(Icons.Default.WifiOff, contentDescription = null,
                    tint = MaterialTheme.colorScheme.onErrorContainer)
                Spacer(Modifier.width(8.dp))
                Text("Sin conexión a internet",
                    color = MaterialTheme.colorScheme.onErrorContainer)
            }
        }
    }
}
```

---

## 🏪 In-App Updates e In-App Reviews

```kotlin
// build.gradle.kts
// implementation("com.google.android.play:app-update-ktx:2.1.0")
// implementation("com.google.android.play:review-ktx:2.0.2")

// In-App Updates
class UpdateManager(private val activity: Activity) {

    private val appUpdateManager = AppUpdateManagerFactory.create(activity)

    // Verificar si hay actualización disponible
    suspend fun checkForUpdate(): AppUpdateInfo? = suspendCancellableCoroutine { cont ->
        appUpdateManager.appUpdateInfo.addOnSuccessListener { info ->
            if (info.updateAvailability() == UpdateAvailability.UPDATE_AVAILABLE) {
                cont.resume(info)
            } else {
                cont.resume(null)
            }
        }.addOnFailureListener { cont.resume(null) }
    }

    // Actualización flexible (en background — recomendada para updates no críticos)
    fun startFlexibleUpdate(info: AppUpdateInfo, launcher: ActivityResultLauncher<IntentSenderRequest>) {
        appUpdateManager.startUpdateFlowForResult(
            info,
            launcher,
            AppUpdateOptions.newBuilder(AppUpdateType.FLEXIBLE).build(),
        )
    }

    // Actualización inmediata (bloquea la app — para updates críticos de seguridad)
    fun startImmediateUpdate(info: AppUpdateInfo, launcher: ActivityResultLauncher<IntentSenderRequest>) {
        appUpdateManager.startUpdateFlowForResult(
            info,
            launcher,
            AppUpdateOptions.newBuilder(AppUpdateType.IMMEDIATE).build(),
        )
    }

    // Completar actualización flexible (instalar después de descargada)
    fun completeFlexibleUpdate() = appUpdateManager.completeUpdate()

    fun registerInstallStateListener(onDownloaded: () -> Unit): InstallStateUpdatedListener {
        val listener = InstallStateUpdatedListener { state ->
            if (state.installStatus() == InstallStatus.DOWNLOADED) onDownloaded()
        }
        appUpdateManager.registerListener(listener)
        return listener
    }

    fun unregisterListener(listener: InstallStateUpdatedListener) =
        appUpdateManager.unregisterListener(listener)
}

// In-App Review
class ReviewManager(private val context: Context) {
    private val reviewManager = ReviewManagerFactory.create(context)

    suspend fun requestReview(activity: Activity) {
        try {
            val reviewInfo = reviewManager.requestReviewFlow().await()
            reviewManager.launchReviewFlow(activity, reviewInfo).await()
            // Nota: Google no garantiza que el dialog se muestre siempre
            // (tiene límites internos de frecuencia)
        } catch (e: Exception) {
            // Falla silenciosamente — nunca mostrar error al usuario por esto
        }
    }
}

// Usar en Composable/ViewModel
@Composable
fun AppUpdateHandler() {
    val context = LocalContext.current
    val activity = context as Activity
    val updateManager = remember { UpdateManager(activity) }

    val updateLauncher = rememberLauncherForActivityResult(
        ActivityResultContracts.StartIntentSenderForResult()
    ) { result ->
        if (result.resultCode == Activity.RESULT_CANCELED) {
            // Usuario canceló — manejar según criticidad
        }
    }

    LaunchedEffect(Unit) {
        val updateInfo = updateManager.checkForUpdate() ?: return@LaunchedEffect
        val priority = updateInfo.updatePriority()

        if (priority >= 4) {
            // Alta prioridad → actualización inmediata
            updateManager.startImmediateUpdate(updateInfo, updateLauncher)
        } else {
            // Baja prioridad → flexible
            updateManager.startFlexibleUpdate(updateInfo, updateLauncher)
        }
    }
}
```

---

## 🏗️ Build Variants y Flavors

```kotlin
// build.gradle.kts (app) — configuración completa

android {
    // Signing configs
    signingConfigs {
        create("release") {
            storeFile = file(System.getenv("KEYSTORE_PATH") ?: "keystore.jks")
            storePassword = System.getenv("KEYSTORE_PASS") ?: ""
            keyAlias = System.getenv("KEY_ALIAS") ?: ""
            keyPassword = System.getenv("KEY_PASS") ?: ""
        }
    }

    buildTypes {
        debug {
            applicationIdSuffix = ".debug"
            versionNameSuffix = "-debug"
            isDebuggable = true
            isMinifyEnabled = false
            buildConfigField("String", "API_BASE_URL", "\"https://dev.api.miapp.com/v1/\"")
            buildConfigField("Boolean", "ENABLE_LOGGING", "true")
            resValue("string", "app_name", "Mi App (Debug)")
        }
        release {
            isMinifyEnabled = true
            isShrinkResources = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
            signingConfig = signingConfigs.getByName("release")
            buildConfigField("String", "API_BASE_URL", "\"https://api.miapp.com/v1/\"")
            buildConfigField("Boolean", "ENABLE_LOGGING", "false")
        }
        create("staging") {
            initWith(getByName("release"))          // Hereda de release
            applicationIdSuffix = ".staging"
            versionNameSuffix = "-staging"
            isDebuggable = true
            buildConfigField("String", "API_BASE_URL", "\"https://staging.api.miapp.com/v1/\"")
            buildConfigField("Boolean", "ENABLE_LOGGING", "true")
        }
    }

    // Product Flavors — variantes del producto
    flavorDimensions += listOf("tier", "store")

    productFlavors {
        create("free") {
            dimension = "tier"
            applicationIdSuffix = ".free"
            buildConfigField("Boolean", "IS_PRO", "false")
            buildConfigField("Int", "MAX_ITEMS", "10")
        }
        create("pro") {
            dimension = "tier"
            applicationIdSuffix = ".pro"
            buildConfigField("Boolean", "IS_PRO", "true")
            buildConfigField("Int", "MAX_ITEMS", "Int.MAX_VALUE")
        }
        create("playStore") {
            dimension = "store"
        }
        create("huaweiStore") {
            dimension = "store"
            buildConfigField("String", "STORE", "\"huawei\"")
        }
    }

    // Habilitar BuildConfig
    buildFeatures {
        buildConfig = true
        compose = true
    }
}
```

```kotlin
// Usar BuildConfig en código
val apiUrl = BuildConfig.API_BASE_URL
val isPro = BuildConfig.IS_PRO
val enableLogging = BuildConfig.ENABLE_LOGGING

// Fuentes de código por flavor: src/free/kotlin/, src/pro/kotlin/
// Fuentes de recursos por flavor: src/free/res/, src/pro/res/
// Fuente compartida sigue en: src/main/kotlin/, src/main/res/
```

---

## 🛡️ ProGuard / R8 — reglas esenciales

```pro
# proguard-rules.pro

# Kotlin
-keep class kotlin.** { *; }
-keepclassmembers class **$WhenMappings { *; }
-keepclassmembers class kotlin.Metadata { *; }

# Kotlinx Serialization
-keepattributes *Annotation*, InnerClasses
-dontnote kotlinx.serialization.AnnotationsKt
-keepclassmembers class kotlinx.serialization.json.** { *** Companion; }
-keepclasseswithmembers class **$$serializer { *; }
@kotlinx.serialization.Serializable class * { *; }

# Retrofit
-keepattributes Signature, Exceptions
-keepclassmembers,allowshrinking,allowobfuscation interface * {
    @retrofit2.http.* <methods>;
}
-dontwarn org.codehaus.mojo.animal_sniffer.IgnoreJRERequirement
-dontwarn javax.annotation.**

# OkHttp
-dontwarn okhttp3.**
-dontwarn okio.**
-keepnames class okhttp3.internal.publicsuffix.PublicSuffixDatabase

# Room
-keep class * extends androidx.room.RoomDatabase
-keep @androidx.room.Entity class *
-dontwarn androidx.room.paging.**

# Koin
-keepclassmembers class * { @org.koin.core.annotation.* <fields>; }

# Firebase
-keep class com.google.firebase.** { *; }
-dontwarn com.google.firebase.**

# Glide / Coil
-keep public class * implements com.bumptech.glide.module.GlideModule
-keep class * extends com.bumptech.glide.AppGlideModule { <init>(...); }

# Parcelable
-keepclassmembers class * implements android.os.Parcelable {
    static ** CREATOR;
}

# Enums
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

# Crashlytics
-keepattributes SourceFile, LineNumberTable
-keep public class * extends java.lang.Exception
-keep class com.google.firebase.crashlytics.** { *; }

# Modelos de datos (si usás reflexión) — ajustar al paquete propio
-keep class com.tuempresa.tuapp.data.model.** { *; }

# Mantener stack traces legibles (para Crashlytics)
-keepattributes SourceFile, LineNumberTable
-renamesourcefileattribute SourceFile
```

