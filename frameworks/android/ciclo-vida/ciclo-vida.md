# Android Jetpack Compose — Ciclo de vida y estado

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

    // Leer parámetro de navegación type-safe (Navigation 3)
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

### Predictive Back — gesto de vuelta animado

Desde Android 13+ el sistema puede mostrar una previsualización animada del destino al hacer el gesto de "atrás" antes de completarlo. `BackHandler` sigue funcionando para interceptar el back sin más, pero para animar la transición en base al progreso del gesto se usa `PredictiveBackHandler`.

```kotlin
// build.gradle.kts — requiere androidx.activity:activity-compose 1.8.0+ (ver base/base.md)

@Composable
fun EditScreen(onBack: () -> Unit) {
    var progress by remember { mutableFloatStateOf(0f) }
    var isPredictiveBack by remember { mutableStateOf(false) }

    PredictiveBackHandler(enabled = true) { backEvent: Flow<BackEventCompat> ->
        try {
            isPredictiveBack = true
            backEvent.collect { event ->
                progress = event.progress   // 0f..1f — usar para animar escala/alpha/offset
            }
            // El usuario completó el gesto — navegar
            onBack()
        } catch (e: CancellationException) {
            // El usuario canceló el gesto a mitad de camino — volver al estado normal
            isPredictiveBack = false
            progress = 0f
        }
    }

    Box(
        modifier = Modifier.graphicsLayer {
            if (isPredictiveBack) {
                scaleX = 1f - (progress * 0.1f)
                scaleY = 1f - (progress * 0.1f)
                alpha = 1f - (progress * 0.3f)
            }
        }
    ) {
        // Contenido de la pantalla
    }
}
```

> Con `targetSdk 34+` (este cheatsheet usa 36 — ver `base/base.md`), predictive back ya está **habilitado por defecto**, sin necesitar tocar el Manifest. Solo hace falta desactivarlo explícitamente con `android:enableOnBackInvokedCallback="false"` si no lo querés. `PredictiveBackHandler` es necesario únicamente cuando querés **animar** la transición vos mismo en base al progreso del gesto — para solo interceptar el back sin animación, `BackHandler` alcanza.

