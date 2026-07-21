# Flutter — Cheatsheet

Cheatsheet de Flutter organizado por temas. Cada carpeta contiene el/los `.md` de su categoría.

---

## 📑 Índice

### Base
- [CLI, estructura de proyecto y punto de entrada](base/base.md)

### Dart moderno
- [Dart 3.x — null safety, records, pattern matching, sealed classes, extensions, freezed](dart-moderno/dart-moderno.md)

### Widgets y UI
- [Widgets fundamentales, layout, listas, widgets visuales, Scaffold, ciclo de vida](widgets/widgets.md)
- [UI avanzada — CustomPainter, Slivers, LayoutBuilder/MediaQuery, InheritedWidget](ui-avanzada/ui-avanzada.md)

### Navegación
- [GoRouter — básico y avanzado (StatefulShellRoute, guards, deep links)](navegacion/navegacion.md)

### Estado
- [BLoC — Cubit, Bloc y su uso en la UI](estado/bloc.md)

### Red y APIs
- [HTTP y Dio — cliente HTTP, interceptores, refresh token](red/http-dio.md)
- [Async — FutureBuilder y StreamBuilder](red/async.md)

### Firebase
- [Setup, Crashlytics, Auth, Firestore, Storage, Cloud Messaging](firebase/firebase.md)

### Persistencia
- [SharedPreferences y Drift](persistencia/drift.md)

### Arquitectura
- [Clean Architecture — capas, repositorios, use cases, get_it](arquitectura/clean-architecture.md)

### Plataforma nativa
- [Platform Channels, notificaciones locales, permisos/cámara/geolocalización, connectivity](plataforma-nativa/plataforma-nativa.md)

### Almacenamiento
- [path_provider, gal, file_picker, share_plus, url_launcher](almacenamiento/almacenamiento.md)

### Background
- [workmanager (tareas periódicas), flutter_background_service (servicio continuo)](background/background.md)

### Accesibilidad
- [Semantics, touch targets, contraste, testing con meetsGuideline](accesibilidad/accesibilidad.md)

### Animaciones
- [Animaciones básicas y avanzadas — flutter_animate, AnimationController](animaciones/animaciones.md)

### Temas y estilos
- [ThemeData, Material 3, diálogos, sheets y snackbars](temas/temas.md)

### Concurrencia
- [Isolates y compute](concurrencia/isolates.md)

### Seguridad
- [flutter_secure_storage, biometría, certificate pinning, ofuscación](seguridad/seguridad.md)

### Internacionalización
- [i18n — ARB files, plurales, fechas, monedas, idioma en runtime](i18n/i18n.md)

### Testing
- [Unit, BLoC, widget y golden tests](testing/testing.md)

### Performance
- [const, RepaintBoundary, KeepAlive, context.select](performance/performance.md)

### Build y distribución
- [Flavors, dart-define-from-file, ofuscación, split APK](build/build.md)

---

## 📦 Paquetes esenciales

| Categoría | Paquete | Descripción |
|---|---|---|
| Estado | `bloc` + `flutter_bloc` | Manejo de estado con BLoC/Cubit |
| Navegación | `go_router` | Router declarativo oficial |
| HTTP | `dio` | Cliente HTTP con interceptores |
| HTTP | `http` | Cliente HTTP simple |
| Serialización | `json_serializable` | Generación de fromJson/toJson |
| Serialización | `freezed` | Clases inmutables + unions |
| Persistencia | `shared_preferences` | Clave-valor simple |
| Persistencia | `drift` | Base de datos SQL type-safe y reactiva |
| Persistencia | `sqflite` | SQLite para Flutter |
| Inyección de dep. | `get_it` | Service locator |
| Imágenes | `cached_network_image` | Caché de imágenes de red |
| Imágenes | `image_picker` | Seleccionar imagen de galería/cámara |
| Auth | `firebase_auth` | Autenticación con Firebase |
| Push | `firebase_messaging` | Notificaciones push |
| Forms | `reactive_forms` | Manejo avanzado de formularios |
| Archivos | `path_provider` | Directorios de la app (docs, cache, soporte) |
| Archivos | `file_picker` | Elegir archivos del dispositivo |
| Archivos | `gal` | Guardar imágenes/videos en la galería |
| Compartir | `share_plus` | Compartir texto, links y archivos |
| Compartir | `url_launcher` | Abrir URLs, teléfono, email, otras apps |
| Background | `workmanager` | Tareas periódicas/diferidas garantizadas por el SO |
| Background | `flutter_background_service` | Servicio en foreground continuo |
| UI | `flutter_svg` | Soporte de SVG |
| UI | `shimmer` | Efecto shimmer de carga |
| UI | `lottie` | Animaciones Lottie |
| Testing | `mocktail` | Mocking para tests |
| Logging | `logger` | Logging con niveles y estilos |
| Env | `flutter_dotenv` | Variables de entorno desde `.env` |

---

## 💡 Buenas prácticas

- **Preferir `const` siempre que sea posible** — reduce reconstrucciones innecesarias.
- **Extraer widgets en lugar de anidar** — métodos privados que devuelven Widget son válidos, pero widgets separados son más reutilizables y mejoran el rebuild.
- **No poner lógica en el `build()`** — moverla a cubits/blocs, use cases o controllers.
- **Siempre disponer los controllers** (`TextEditingController`, `AnimationController`, etc.) en `dispose()`.
- **Usar `ListView.builder` para listas largas** — nunca `ListView` con children fijos para listas dinámicas.
- **Nombrar `key` en listas dinámicas** — `ValueKey`, `ObjectKey` para que Flutter identifique elementos correctamente.
- **Separar la UI de la lógica** — Feature Folder o Clean Architecture según la escala del proyecto.
- **`context.read()` para acciones, `context.watch()` para estado** — no usar `watch` fuera del `build`.
- **Testear la lógica de negocio** con unit tests, no la UI en primer lugar.
- **Usar `flutter analyze` y `dart fix`** regularmente para mantener el código limpio.
