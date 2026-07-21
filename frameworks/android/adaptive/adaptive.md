# Android Jetpack Compose — Adaptive layouts

---

## 📏 WindowSizeClass — adaptar UI al tamaño disponible

```kotlin
// build.gradle.kts
// implementation("androidx.compose.material3.adaptive:adaptive:1.2.0")

import androidx.compose.material3.adaptive.currentWindowAdaptiveInfo

@Composable
fun MyApp() {
    // El tamaño se recalcula automáticamente al rotar, en multi-window, o al plegar/desplegar
    val windowSizeClass = currentWindowAdaptiveInfo(supportLargeAndXLargeWidth = true).windowSizeClass

    val showTopAppBar = windowSizeClass.isHeightAtLeastBreakpoint(
        WindowSizeClass.HEIGHT_DP_MEDIUM_LOWER_BOUND
    )

    MyScreen(showTopAppBar = showTopAppBar)
}
```

| Ancho | Breakpoint | Ejemplo típico |
|---|---|---|
| Compact | < 600dp | Teléfono en portrait |
| Medium | 600dp – 840dp | Tablet en portrait, teléfono plegable abierto |
| Expanded | 840dp – 1200dp | Tablet en landscape |
| Large | 1200dp – 1600dp | Tablet grande, foldable desplegado |
| Extra-large | ≥ 1600dp | Pantalla de escritorio (ChromeOS) |

> `WindowSizeClass` se determina por el **espacio disponible de la ventana**, no por el tipo físico de dispositivo — la mayoría de las decisiones de layout deberían basarse en el ancho.

---

## 🧭 NavigationSuiteScaffold — navegación que se adapta sola

```kotlin
// build.gradle.kts
// implementation("androidx.compose.material3:material3-adaptive-navigation-suite:1.5.0-alpha24")
// (todavía no tiene release estable propio — trackear versión en Compose Material3 releases)

enum class AppDestinations(@StringRes val label: Int, val icon: ImageVector) {
    HOME(R.string.home, Icons.Default.Home),
    FAVORITES(R.string.favorites, Icons.Default.Favorite),
    PROFILE(R.string.profile, Icons.Default.AccountBox),
}

@Composable
fun MyApp() {
    var currentDestination by rememberSaveable { mutableStateOf(AppDestinations.HOME) }

    NavigationSuiteScaffold(
        navigationSuiteItems = {
            AppDestinations.entries.forEach { destination ->
                item(
                    icon = { Icon(destination.icon, contentDescription = stringResource(destination.label)) },
                    label = { Text(stringResource(destination.label)) },
                    selected = destination == currentDestination,
                    onClick = { currentDestination = destination },
                )
            }
        },
    ) {
        // Contenido de la pantalla actual
        when (currentDestination) {
            AppDestinations.HOME -> HomeScreen()
            AppDestinations.FAVORITES -> FavoritesScreen()
            AppDestinations.PROFILE -> ProfileScreen()
        }
    }
}
```

`NavigationSuiteScaffold` elige automáticamente el componente de navegación según el tamaño de ventana:

| Tamaño | Navegación usada |
|---|---|
| Compact | `NavigationBar` (bottom bar) |
| Medium / Expanded | `NavigationRail` |
| Expanded grande / preferencia explícita | `NavigationDrawer` |

```kotlin
// Forzar un tipo de navegación específico en vez del automático
val adaptiveInfo = currentWindowAdaptiveInfo()
val navSuiteType = if (adaptiveInfo.windowSizeClass.isWidthAtLeastBreakpoint(WIDTH_DP_EXPANDED_LOWER_BOUND)) {
    NavigationSuiteType.NavigationDrawer
} else {
    NavigationSuiteScaffoldDefaults.calculateFromAdaptiveInfo(adaptiveInfo)
}

NavigationSuiteScaffold(
    navigationSuiteItems = { /* ... */ },
    layoutType = navSuiteType,
) { /* ... */ }
```

---

## 📱 Layouts de dos paneles (lista-detalle)

```kotlin
// build.gradle.kts
// implementation("androidx.compose.material3.adaptive:adaptive-layout:1.2.0")
// implementation("androidx.compose.material3.adaptive:adaptive-navigation:1.2.0")

import androidx.compose.material3.adaptive.ExperimentalMaterial3AdaptiveApi
import androidx.compose.material3.adaptive.layout.AnimatedPane
import androidx.compose.material3.adaptive.layout.ListDetailPaneScaffoldRole
import androidx.compose.material3.adaptive.navigation.NavigableListDetailPaneScaffold
import androidx.compose.material3.adaptive.navigation.rememberListDetailPaneScaffoldNavigator

// NavigableListDetailPaneScaffold — muestra lista+detalle lado a lado en pantallas anchas,
// y solo un panel a la vez (con back automático) en pantallas compactas
@OptIn(ExperimentalMaterial3AdaptiveApi::class)
@Composable
fun ProductListDetail() {
    val navigator = rememberListDetailPaneScaffoldNavigator<Product>()
    val scope = rememberCoroutineScope()

    NavigableListDetailPaneScaffold(
        navigator = navigator,
        listPane = {
            AnimatedPane {
                ProductListScreen(
                    onProductClick = { product ->
                        scope.launch { navigator.navigateTo(ListDetailPaneScaffoldRole.Detail, product) }
                    }
                )
            }
        },
        detailPane = {
            AnimatedPane {
                navigator.currentDestination?.contentKey?.let { product ->
                    ProductDetailScreen(
                        product = product,
                        onBack = { scope.launch { navigator.navigateBack() } },
                    )
                }
            }
        },
    )
}
```

---

## 📂 Foldables — postura del dispositivo

```kotlin
// build.gradle.kts
// implementation("androidx.window:window:1.4.0")

// Detectar el pliegue (hinge) para no poner contenido importante debajo
@Composable
fun rememberFoldingFeature(): FoldingFeature? {
    val context = LocalContext.current
    var foldingFeature by remember { mutableStateOf<FoldingFeature?>(null) }

    LaunchedEffect(Unit) {
        WindowInfoTracker.getOrCreate(context)
            .windowLayoutInfo(context as Activity)
            .collect { layoutInfo ->
                foldingFeature = layoutInfo.displayFeatures
                    .filterIsInstance<FoldingFeature>()
                    .firstOrNull()
            }
    }
    return foldingFeature
}

// Adaptar el layout si el dispositivo está en postura "tabletop" o "book" (medio plegado)
val isTableTop = foldingFeature?.state == FoldingFeature.State.HALF_OPENED &&
    foldingFeature?.orientation == FoldingFeature.Orientation.HORIZONTAL
```
