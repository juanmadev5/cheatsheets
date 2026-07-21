# Android Jetpack Compose — Interop con Views

---

> Esto es para **interop y migración incremental** — pantallas que todavía usan el sistema de Views clásico (XML, Fragments) conviviendo con Compose. No es una guía del sistema de Views en sí.

## 🧩 AndroidView — usar una View clásica dentro de Compose

```kotlin
// Envolver una View que no tiene equivalente en Compose (ej: un SDK de terceros con su propia View)
@Composable
fun MapViewContainer(cameraPosition: LatLng) {
    AndroidView(
        modifier = Modifier.fillMaxSize(),
        factory = { context ->
            // factory: se llama UNA sola vez, para construir la View
            MapView(context).apply {
                onCreate(null)
                getMapAsync { map -> map.moveCamera(CameraUpdateFactory.newLatLng(cameraPosition)) }
            }
        },
        update = { mapView ->
            // update: se llama cada vez que cambia el estado de Compose — mantenerlo liviano
            mapView.getMapAsync { map -> map.moveCamera(CameraUpdateFactory.newLatLng(cameraPosition)) }
        },
        onRelease = { mapView -> mapView.onDestroy() },
    )
}

// Dentro de una LazyColumn — usar onReset para reciclar la View en vez de recrearla
LazyColumn {
    items(100) { index ->
        AndroidView(
            factory = { context -> LegacyItemView(context) },
            update = { view -> view.bind(items[index]) },
            onReset = { view -> view.clear() },   // Habilita reuso de la View al hacer scroll
        )
    }
}
```

---

## 📐 AndroidViewBinding — embeber un layout XML completo

```kotlin
// build.gradle.kts — requiere view binding habilitado
// android { buildFeatures { viewBinding = true } }

// Útil durante una migración incremental cuando ya existe un layout XML complejo
// (formulario, layout con muchas dependencias) que todavía no vale la pena reescribir
@Composable
fun LegacyFormScreen() {
    AndroidViewBinding(FormLayoutBinding::inflate) {
        // Acceso directo a las Views del binding generado
        submitButton.setOnClickListener { /* ... */ }
        emailInput.doAfterTextChanged { /* ... */ }
    }
}
```

---

## 🧭 AndroidFragment — embeber un Fragment existente

```kotlin
// build.gradle.kts
// implementation("androidx.fragment:fragment-compose:1.8.9")

// Solo como paso transitorio durante migración — el Fragment sigue siendo Fragment,
// no se vuelve más "Compose" por estar embebido
@Composable
fun ScreenWithLegacyFragment() {
    AndroidFragment<LegacyCheckoutFragment>(
        modifier = Modifier.fillMaxSize(),
        arguments = bundleOf("orderId" to orderId),
    )
}
```

---

## 🎨 ComposeView — usar Compose dentro de una Activity/Fragment con Views

```kotlin
// Caso típico: Activity o Fragment legacy que quiere renderizar UNA pantalla nueva en Compose
// sin migrar toda la navegación todavía

class LegacyFragment : Fragment() {

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?,
    ): View = ComposeView(requireContext()).apply {
        // Evita fugas de memoria — dispone la composición cuando se destruye la View del Fragment
        setViewCompositionStrategy(ViewCompositionStrategy.DisposeOnViewTreeLifecycleDestroyed)
        setContent {
            MyAppTheme {
                ProductDetailScreen(productId = arguments?.getString("productId"))
            }
        }
    }
}

// Insertar un ComposeView dentro de un layout XML existente
// <androidx.compose.ui.platform.ComposeView
//     android:id="@+id/compose_view"
//     android:layout_width="match_parent"
//     android:layout_height="wrap_content" />

binding.composeView.apply {
    setViewCompositionStrategy(ViewCompositionStrategy.DisposeOnViewTreeLifecycleDestroyed)
    setContent { MyAppTheme { HeaderBanner() } }
}
```

| Estrategia de composición | Cuándo usarla |
|---|---|
| `DisposeOnViewTreeLifecycleDestroyed` | Fragments y Views cuyo lifecycle no se conoce de antemano (caso más común) |
| `DisposeOnDetachedFromWindowOrReleasedFromPool` | Default — items de `RecyclerView` u otras Views reciclables |
| `DisposeOnLifecycleDestroyed(lifecycle)` | Cuando tenés una referencia explícita al `Lifecycle` |

---

## ↔️ Comunicación entre Compose y Views

```kotlin
// Compose → View: mediante el parámetro `update` de AndroidView (ver arriba)

// View → Compose: exponer un callback/listener desde la View y capturarlo en `factory`
AndroidView(
    factory = { context ->
        LegacySignaturePad(context).apply {
            onSignatureCaptured = { bitmap -> onSignatureReady(bitmap) }  // Lambda de Compose
        }
    },
)

// Fragment → Compose: usar un ViewModel compartido (scope de Activity) en vez de callbacks directos
val sharedViewModel: CheckoutViewModel = viewModel(viewModelStoreOwner = requireActivity())
```
