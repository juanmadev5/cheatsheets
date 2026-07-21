# Android Jetpack Compose — Navegación

---

## 🧭 Navigation 3

> Navigation 3 es la librería oficial de navegación para Compose, estable desde noviembre de 2025. A diferencia del sistema anterior basado en grafo, **vos sos dueño del back stack** — es una simple `List` de claves que podés manipular directamente.

### Dependencias

```kotlin
// build.gradle.kts
dependencies {
    val nav3Version = "1.1.4"
    implementation("androidx.navigation3:navigation3-runtime:$nav3Version")
    implementation("androidx.navigation3:navigation3-ui:$nav3Version")
    implementation("androidx.lifecycle:lifecycle-viewmodel-navigation3:2.11.0")

    // Para rememberNavBackStack con persistencia
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.11.0")
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
implementation("androidx.lifecycle:lifecycle-viewmodel-navigation3:2.11.0")

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
