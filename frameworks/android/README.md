# Android Jetpack Compose — Cheatsheet

Cheatsheet de Android Jetpack Compose organizado por temas. Cada carpeta contiene el/los `.md` de su categoría.

---

## 📑 Índice

### Base
- [Gradle, estructura de proyecto y punto de entrada](base/base.md)

### Composables y UI
- [Composables fundamentales, layout, modifier y widgets visuales](composables/composables.md)

### UI avanzada
- [PullToRefresh, SwipeToDismiss, ModalBottomSheet/DatePicker/SearchBar, Animaciones](ui-avanzada/ui-avanzada.md)

### Interop con Views
- [AndroidView, AndroidViewBinding, AndroidFragment y ComposeView — interop y migración incremental](interop/interop.md)

### Adaptive layouts
- [WindowSizeClass, NavigationSuiteScaffold, layouts lista-detalle y foldables](adaptive/adaptive.md)

### Navegación
- [Navigation 3 — back stack propio, entryProvider, NavDisplay](navegacion/navegacion.md)

### Estado y arquitectura
- [ViewModel + StateFlow y Koin (inyección de dependencias)](estado/estado.md)

### Arquitectura
- [Clean Architecture — capas, Repository, UseCase, Koin](arquitectura/clean-architecture.md)

### Base de datos y persistencia
- [Room, DataStore y Paging 3](persistencia/persistencia.md)

### Red y APIs
- [Retrofit, DTOs con kotlinx.serialization, manejo de errores y OkHttp](red/retrofit.md)

### Coroutines y Flow
- [Coroutines, Flow y StateFlow vs SharedFlow](coroutines/coroutines.md)

### Firebase
- [Setup, Auth, Firestore, Storage, Crashlytics y Remote Config](firebase/firebase.md)

### Seguridad
- [EncryptedSharedPreferences, Keystore, BiometricPrompt y Network Security Config](seguridad/seguridad.md)

### Background y servicios
- [WorkManager, Foreground Service y Broadcast Receivers](background/background.md)

### Notificaciones
- [Canales, PendingIntent, acciones y badges](notificaciones/notificaciones.md)

### Ciclo de vida y estado
- [rememberSaveable, SavedStateHandle, BackHandler y Predictive Back](ciclo-vida/ciclo-vida.md)

### Permisos y Manifest
- [AndroidManifest.xml, tipos de permisos y permisos en runtime](permisos/permisos.md)

### Intents
- [Intent explícito/implícito, flags, PendingIntent y Deep Links](intents/intents.md)

### Almacenamiento
- [MediaStore, SAF, FileProvider y almacenamiento interno/externo](almacenamiento/almacenamiento.md)

### Temas
- [ThemeData con Material 3 y Splash Screen API](temas/temas.md)

### Performance
- [derivedStateOf, @Stable/@Immutable, CompositionLocal y recomposición](performance/performance.md)

### Accesibilidad
- [Semantics, mergeDescendants, CustomAction y LiveRegion](accesibilidad/accesibilidad.md)

### Internacionalización
- [strings.xml, plurales, formateo con locale e idioma en runtime](i18n/i18n.md)

### Widgets de pantalla de inicio
- [App Widget con Glance](widgets-inicio/glance.md)

### Connectivity
- [Monitoreo de red con NetworkCallback como Flow](connectivity/connectivity.md)

### Play Store
- [In-App Updates e In-App Reviews](play-store/play-store.md)

### Build y distribución
- [Build Variants, Flavors y reglas ProGuard/R8](build/build.md)

### Testing
- [Unit, ViewModel y testing con coroutines](testing/testing.md)

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
