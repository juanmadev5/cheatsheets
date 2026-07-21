# Android Jetpack Compose — Performance

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
// implementation("org.jetbrains.kotlinx:kotlinx-collections-immutable:0.5.1")
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

