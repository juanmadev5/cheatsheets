# Sources

Registro de documentación oficial consultada para verificar y actualizar el contenido de los cheatsheets. Se agrega una sección por framework a medida que se investiga/actualiza su contenido — no implica que todo el cheatsheet haya sido verificado, solo lo que efectivamente se contrastó contra estas fuentes.

**Regla:** solo se listan fuentes oficiales — documentación del framework/librería, release notes o repositorio oficial del proyecto (developer.android.com, kotlinlang.org, el repo GitHub del propio proyecto, etc.). Nada de blogs, Medium ni artículos de terceros.

---

## Android Jetpack Compose

_Investigado: julio 2026 (actualización de versiones y remoción de contenido deprecado)._

### Kotlin / AGP / SDK
- https://kotlinlang.org/docs/releases.html — release process y versión estable actual de Kotlin
- https://blog.jetbrains.com/kotlin/2026/03/kotlin-2-3-20-released/ — Kotlin 2.3.20
- https://developer.android.com/about/versions/16/setup-sdk — SDK de Android 16 (API 36)
- https://developer.android.com/google/play/requirements/target-sdk — requisito de targetSdk de Google Play
- https://developer.android.com/build/jdks — versión de Java usada en builds de Android

### Jetpack Compose / Material 3
- https://developer.android.com/develop/ui/compose/bom — uso del Compose BOM
- https://developer.android.com/develop/ui/compose/bom/bom-mapping — mapeo BOM → versiones de `compose-ui` / `compose-material3`
- https://developer.android.com/jetpack/androidx/releases/compose — release notes de Compose
- https://developer.android.com/blog/posts/whats-new-in-the-jetpack-compose-april-26-release — novedades Compose abril 2026

### Navigation 3
- https://developer.android.com/guide/navigation/navigation-3 — overview
- https://developer.android.com/guide/navigation/navigation-3/basics — API básica (entry, entryProvider, NavDisplay)
- https://developer.android.com/jetpack/androidx/releases/navigation3 — versión estable (1.1.4) y artifacts

### AndroidX (Activity, Lifecycle, Room, DataStore, WorkManager, Paging, Credentials, Security, Biometric, Glance)
- https://developer.android.com/jetpack/androidx/releases/activity
- https://developer.android.com/jetpack/androidx/releases/lifecycle
- https://developer.android.com/jetpack/androidx/releases/room — estado de Room 2.x (mantenimiento) vs Room 3 (en desarrollo)
- https://developer.android.com/jetpack/androidx/releases/datastore
- https://developer.android.com/jetpack/androidx/releases/work
- https://developer.android.com/jetpack/androidx/releases/paging
- https://developer.android.com/jetpack/androidx/releases/credentials
- https://developer.android.com/jetpack/androidx/releases/security — deprecación de `EncryptedSharedPreferences`/`MasterKey`
- https://developer.android.com/jetpack/androidx/releases/biometric
- https://developer.android.com/jetpack/androidx/releases/glance

### Almacenamiento seguro (reemplazo de EncryptedSharedPreferences)
- https://developer.android.com/jetpack/androidx/releases/security — confirma la deprecación de `EncryptedSharedPreferences`/`MasterKey` a favor de Android Keystore
- https://github.com/google/tink — repositorio oficial de Tink (Google), usado para el patrón DataStore + Tink + Android Keystore

### Koin
- https://insert-koin.io/docs/setup/koin/ — setup con BOM, versión estable

### Retrofit / OkHttp
- https://github.com/square/retrofit/blob/trunk/CHANGELOG.md — Retrofit 3.0.0
- https://github.com/square/okhttp — versión estable de OkHttp

### Kotlinx (Coroutines, Serialization, Collections)
- https://github.com/Kotlin/kotlinx.coroutines/releases
- https://github.com/Kotlin/kotlinx.serialization/releases
- https://github.com/Kotlin/kotlinx.collections.immutable

### Coil
- https://github.com/coil-kt/coil/releases
- https://coil-kt.github.io/coil/upgrading_to_coil3/ — migración a Coil 3 (nuevo groupId, `coil-network-okhttp` separado)

### Firebase
- https://firebase.google.com/support/release-notes/android — versión del BoM y remoción de módulos `-ktx` (integrados al módulo principal desde jul. 2025)

### Play Core (In-App Updates / Reviews)
- https://developer.android.com/reference/com/google/android/play/core/release-notes
- https://developer.android.com/reference/com/google/android/play/core/release-notes-in_app_updates

### Interop Compose ↔ Views (tema nuevo)
- https://developer.android.com/develop/ui/compose/migrate/interoperability-apis/views-in-compose — `AndroidView`, `AndroidViewBinding`, `AndroidView` con reciclado (`onReset`) en listas
- https://developer.android.com/develop/ui/compose/migrate/interoperability-apis/compose-in-views — `ComposeView`, `ViewCompositionStrategy`
- https://developer.android.com/jetpack/androidx/releases/fragment — versión de `fragment-compose` (1.8.9) y `AndroidFragment`

### Adaptive layouts (tema nuevo)
- https://developer.android.com/develop/ui/compose/layouts/adaptive/window-size-classes — `currentWindowAdaptiveInfo()`, breakpoints de `WindowSizeClass`
- https://developer.android.com/develop/adaptive-apps/guides/build-adaptive-navigation — `NavigationSuiteScaffold`, dependencia y ejemplo completo
- https://developer.android.com/jetpack/androidx/releases/compose-material3-adaptive — versión estable de `adaptive`/`adaptive-layout`/`adaptive-navigation` (1.2.0) y estado alpha de `material3-adaptive-navigation-suite`
- https://developer.android.com/develop/ui/compose/layouts/adaptive/list-detail — `NavigableListDetailPaneScaffold`, `rememberListDetailPaneScaffoldNavigator`, `ListDetailPaneScaffoldRole`
- https://developer.android.com/jetpack/androidx/releases/window — versión de `androidx.window` (1.5.1) para detección de foldables

### Predictive Back (agregado a ciclo-vida)
- https://developer.android.com/guide/navigation/custom-back/predictive-back-gesture — `PredictiveBackHandler`, `BackEventCompat`, comportamiento default-on con targetSdk 34+

### Splash Screen API (agregado a temas)
- https://developer.android.com/develop/ui/views/launch/splash-screen — setup de tema, `installSplashScreen()`, `setKeepOnScreenCondition`
- https://developer.android.com/jetpack/androidx/releases/core — versión de `core-splashscreen` (1.2.0)

### Arquitectura — Clean Architecture (tema nuevo)
- https://developer.android.com/topic/architecture — capas UI/domain/data, Repository, UseCase, Unidirectional Data Flow (el patrón de DI del ejemplo oficial usa Hilt; se adaptó a Koin para mantener consistencia con el resto del cheatsheet, que ya usa Koin en todos lados)

---

## ASP.NET Core

_Investigado: julio 2026 (actualización de versiones y remoción de contenido deprecado)._

### .NET / SDK
- https://github.com/dotnet/core/blob/main/release-notes/11.0/README.md — .NET 11 (STS) recién programado para nov. 2026, todavía no lanzado
- https://dotnet.microsoft.com/en-us/download/dotnet/11.0 — confirma que .NET 11 sigue en preview
- https://learn.microsoft.com/en-us/dotnet/core/docker/container-images — esquema de tags `mcr.microsoft.com/dotnet/sdk`/`aspnet`

### Paquetes NuGet (versiones)
- https://www.nuget.org/packages/Microsoft.EntityFrameworkCore
- https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.JwtBearer
- https://www.nuget.org/packages/Microsoft.AspNetCore.Identity.EntityFrameworkCore
- https://www.nuget.org/packages/Microsoft.Extensions.Caching.StackExchangeRedis
- https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Testing
- https://www.nuget.org/packages/Npgsql.EntityFrameworkCore.PostgreSQL
- https://www.nuget.org/packages/Mapster
- https://www.nuget.org/packages/Quartz.Extensions.Hosting
- https://www.nuget.org/packages/MailKit
- https://www.nuget.org/packages/Testcontainers.PostgreSql
- https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks/releases — corrige un pin incorrecto de HealthChecks (no sigue el versionado de .NET)

### OpenAPI (Swashbuckle → generador integrado)
- https://learn.microsoft.com/en-us/aspnet/core/fundamentals/openapi/aspnetcore-openapi?view=aspnetcore-10.0 — `Microsoft.AspNetCore.OpenApi` reemplaza a Swashbuckle en el template desde .NET 9
- https://learn.microsoft.com/en-us/aspnet/core/fundamentals/openapi/using-openapi-documents?view=aspnetcore-10.0 — Scalar (`Scalar.AspNetCore`) como UI recomendada
- https://learn.microsoft.com/en-us/aspnet/core/fundamentals/openapi/customize-openapi?view=aspnetcore-10.0
- https://learn.microsoft.com/en-us/aspnet/core/breaking-changes/10/withopenapi-deprecated?view=aspnetcore-10.0 — `.WithOpenApi()` deprecado en .NET 10
- https://www.nuget.org/packages/Microsoft.AspNetCore.OpenApi
- https://www.nuget.org/packages/Scalar.AspNetCore

### FluentValidation
- https://docs.fluentvalidation.net/en/latest/aspnet.html — el pipeline automático de `FluentValidation.AspNetCore` ya no se recomienda (no es async, no soporta Minimal APIs); reemplazado por `IValidator<T>.ValidateAsync()` manual
- https://www.nuget.org/packages/FluentValidation.AspNetCore — página marcada como legacy/deprecado
- https://www.nuget.org/packages/FluentValidation

### Licencias (MediatR / AutoMapper / FluentAssertions)
- https://github.com/LuckyPennySoftware/MediatR — repo oficial, modelo de licencia comercial
- https://mediatr.io — tier Community gratis para orgs con &lt;$5M USD de facturación anual
- https://www.nuget.org/packages/AutoMapper
- https://automapper.io — mismos términos de licencia que MediatR
- https://www.nuget.org/packages/FluentAssertions — confirma que v8+ requiere licencia comercial (Xceed); se fija la última 7.x libre (Apache 2.0)

### Testing
- https://github.com/xunit/xunit/blob/main/src/xunit.v3.core/IAsyncLifetime.cs — `IAsyncLifetime` ahora usa `ValueTask` (cambio real de v2 a v3)
- https://xunit.net/docs/getting-started/v3/migration — requiere `<OutputType>Exe</OutputType>`, runtime mínimo .NET 8
- https://www.nuget.org/packages/xunit.v3
- https://www.nuget.org/packages/xunit.runner.visualstudio
- https://www.nuget.org/packages/Moq — confirma que no hubo cambios necesarios

---

## Flutter

_Investigado: julio 2026 (actualización de versiones y remoción de contenido deprecado)._

### Flutter / Dart SDK
- https://docs.flutter.dev/release/archive — versión estable actual
- https://storage.googleapis.com/flutter_infra_release/releases/releases_linux.json — versión exacta (Flutter 3.44.7 / Dart 3.12.2)
- https://dart.dev/get-dart
- https://docs.flutter.dev/release/whats-new

### Material 3 / Color API
- https://api.flutter.dev/flutter/material/ThemeData/useMaterial3.html — `useMaterial3` sigue siendo válido (default `true`, no deprecado)
- https://api.flutter.dev/flutter/dart-ui/Color/withOpacity.html — `withOpacity()` deprecado a favor de `withValues(alpha:)`

### Concurrencia
- https://dart.dev/language/concurrency — `Isolate.run()` sigue siendo la API recomendada; `compute()` (wrapper de Flutter) sigue vigente

### freezed / json_serializable
- https://pub.dev/packages/freezed
- https://pub.dev/packages/freezed_annotation
- https://pub.dev/packages/json_serializable
- https://pub.dev/packages/json_annotation
- https://github.com/rrousselGit/freezed/blob/master/packages/freezed/migration_guide.md — breaking change 3.0: requiere `abstract class`/`sealed class` en vez de `class` plana

### go_router / flutter_bloc
- https://pub.dev/packages/go_router
- https://pub.dev/packages/flutter_bloc
- https://pub.dev/packages/bloc

### Drift
- https://pub.dev/packages/drift
- https://pub.dev/packages/drift_flutter
- https://pub.dev/packages/drift_dev
- https://pub.dev/packages/build_runner — confirma `dart run build_runner` como comando canónico (no `flutter pub run`)

### Seguridad y permisos
- https://pub.dev/packages/flutter_secure_storage — confirma que `encryptedSharedPreferences` quedó obsoleto (cifrado por defecto más fuerte desde v10)
- https://github.com/juliansteenbakker/flutter_secure_storage
- https://pub.dev/packages/local_auth
- https://pub.dev/packages/permission_handler — confirma deprecación/remoción de `Permission.storage` en Android 13+ a favor de `photos`/`videos`/`audio`

### Firebase / FlutterFire y otros paquetes
- https://pub.dev/packages/firebase_core, firebase_auth, cloud_firestore, firebase_storage, firebase_messaging, firebase_crashlytics, firebase_analytics
- https://pub.dev/packages/google_sign_in — API v7 (`GoogleSignIn.instance` + `.initialize()`/`.authenticate()`)
- https://pub.dev/packages/sign_in_with_apple
- https://firebase.google.com/docs/flutter/setup
- https://pub.dev/packages/http, dio, pretty_dio_logger
- https://pub.dev/packages/mocktail, bloc_test, patrol, alchemist
- https://pub.dev/packages/intl — bump de versión
- https://pub.dev/packages/flutter_animate, cached_network_image

### Accesibilidad (tema nuevo)
- https://docs.flutter.dev/ui/accessibility/accessibility-testing — testing con `meetsGuideline`, `androidTapTargetGuideline`, `iOSTapTargetGuideline`, `textContrastGuideline`, `labeledTapTargetGuideline`
- https://api.flutter.dev/flutter/flutter_test/meetsGuideline.html

### Almacenamiento y compartir archivos (tema nuevo)
- https://pub.dev/packages/path_provider — 2.1.6, funciones de directorios disponibles
- https://pub.dev/packages/gal — 2.3.2, API `putImage`/`putImageBytes`/`putVideo`
- https://pub.dev/packages/file_picker — 11.0.2 estable, confirma `FilePicker.platform.pickFiles()`
- https://pub.dev/packages/share_plus — 13.2.1, confirma la migración de `Share.share()` (deprecado) a `SharePlus.instance.share(ShareParams(...))`
- https://pub.dev/packages/url_launcher — 6.3.2

### Tareas en background (tema nuevo)
- https://pub.dev/documentation/workmanager/latest/workmanager/Workmanager-class.html — firma exacta de `initialize`, `registerPeriodicTask`, `registerOneOffTask`, `cancelByUniqueName`, `cancelAll`
- https://pub.dev/packages/workmanager — 0.9.0+3
- https://pub.dev/packages/flutter_background_service — 5.1.0, patrón `configure`/`onStart`/`ServiceInstance`

## Spring Boot

_Investigado: julio 2026 (actualización de versiones y remoción de contenido deprecado). Spring Boot 4 salió en nov. 2025 — fue la actualización más grande de las cuatro._

### Versión de Spring Boot / Java LTS
- https://spring.io/blog/2025/11/20/spring-boot-4-0-0-available-now/ — Spring Boot 4.0.0 GA, basado en Spring Framework 7, soporte de primera clase para Java 25 manteniendo compatibilidad con Java 17
- https://docs.spring.io/spring-boot/system-requirements.html — 4.1.0 requiere Java 17 como mínimo, compatible hasta Java 26; Maven 3.6.3+
- https://github.com/spring-projects/spring-boot/releases — confirma 4.1.0 como última estable (10 jun. 2026)

### Cambios de ruptura en Spring Boot 4 (starters, Maven plugin, testing)
- https://docs.spring.io/spring-boot/reference/using/build-systems.html — renombres de starters: `spring-boot-starter-web` → `spring-boot-starter-webmvc` (deprecado el viejo nombre), `spring-boot-starter-flyway` y `spring-boot-starter-security-test` nuevos
- https://docs.spring.io/spring-boot/maven-plugin/using.html — dependencias `optional` excluidas del JAR final por defecto desde el plugin 3.5.7
- https://docs.spring.io/spring-boot/reference/testing/spring-boot-applications.html — `@MockitoBean` reemplaza a `@MockBean`; `@SpringBootTest` ya no autoconfigura `TestRestTemplate`
- https://docs.spring.io/spring-boot/reference/testing/testcontainers.html — `@ServiceConnection` como patrón recomendado actual
- https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide — checklist cruzado contra la documentación primaria

### Jackson 3
- https://spring.io/blog/2025/10/07/introducing-jackson-3-support-in-spring/ — anuncio oficial: Jackson 3 es la librería preferida/default en Spring Boot 4; `JsonMapper` (inmutable) reemplaza a `ObjectMapper`
- https://docs.spring.io/spring-boot/reference/features/json.html — confirma soporte de Jackson 2 deprecado
- https://docs.spring.io/spring-boot/appendix/application-properties/index.html — confirma que las propiedades `spring.jackson.*` existentes siguen siendo válidas bajo Jackson 3
- https://docs.spring.io/spring-boot/api/java/org/springframework/boot/jackson/autoconfigure/JsonMapperBuilderCustomizer.html

### Spring Security
- https://spring.io/blog/2025/11/17/spring-security-releases/ — Spring Security 7.0.0 junto con Spring Boot 4

### MapStruct / Lombok / JWT
- https://github.com/mapstruct/mapstruct/releases — 1.6.3 sigue siendo la última estable (1.7.0 solo en Beta)
- https://projectlombok.org/changelog — 1.18.46 (22 abr. 2026) es la última estable
- https://github.com/jwtk/jjwt/releases — 0.13.0, cambio no rupturista

### Flyway / Testcontainers
- https://docs.spring.io/spring-boot/how-to/data-initialization.html — `spring-boot-starter-flyway` ahora requerido para autoconfiguración
- https://docs.spring.io/spring-boot/appendix/dependency-versions/coordinates.html — versiones gestionadas por Spring Boot 4.1.0 (Flyway 12.4.0, Testcontainers 2.0.5)
- https://github.com/testcontainers/testcontainers-java/releases/tag/2.0.0 — 2.0.0: artifacts renombrados con prefijo `testcontainers-`, soporte JUnit 4 removido
- https://java.testcontainers.org/modules/databases/postgres/ y /quickstart/junit_5_quickstart/ — artifactIds exactos

### Docker
- https://docs.spring.io/spring-boot/reference/packaging/container-images/dockerfiles.html — `-Djarmode=layertools` reemplazado por `-Djarmode=tools ... extract --layers`; la capa `application` extraída ya es un jar ejecutable directo

---

## Dart

_Investigado: julio 2026 (verificación de vigencia del cheatsheet — sin cambio de estructura de carpetas, se mantiene como archivo único en `programming-languages/`)._

- https://dart.dev/get-dart — versión estable actual (Dart 3.12.1)
- https://dart.dev/changelog — historial de releases 3.10 (12 nov. 2025), 3.11 (11 feb. 2026) y 3.12 (18 may. 2026)
- https://dart.dev/language/branches — corrige un error del cheatsheet: Dart **no** tiene `if` como expresión (solo el operador ternario, if-case y switch expressions); también confirma que los `case` no vacíos de un switch *statement* ya no requieren `break` explícito desde Dart 3 (saltan al final del switch automáticamente)
- https://dart.dev/language/dot-shorthands — sintaxis de dot shorthands (`.valor` en vez de `Tipo.valor`), requiere Dart 3.10+
- https://dart.dev/language/constructors — named parameters privados como initializing formals (`required this._campo`), requiere Dart 3.12+
- https://dart.dev/language/built-in-types — separador de dígitos con `_` en literales numéricos, introducido en Dart 3.6
- https://dart.dev/tools/dart-compile — confirma que `dart compile wasm` sigue en desarrollo/experimental (no se agregó al cheatsheet por esa razón)
- https://dart.dev/language/extension-types y https://dart.dev/blog/new-in-dart-3-3-extension-types-javascript-interop-and-more — extension types (Dart 3.3+, feb. 2024): sintaxis, `implements` para variantes transparentes, borrado en tiempo de compilación; tema agregado al cheatsheet tras detectar que faltaba

---

## Kotlin

_Investigado: julio 2026 (verificación de vigencia del cheatsheet — sin cambio de estructura de carpetas, se mantiene como archivo único en `programming-languages/`)._

### Lenguaje
- https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/capitalize.html — corrige un bug real del cheatsheet: `String.capitalize()`/`decapitalize()` fueron deprecados en Kotlin 1.5 y son **error de compilación** desde Kotlin 2.1; el reemplazo oficial es `replaceFirstChar { ... }`
- https://kotlinlang.org/docs/whatsnew21.html — guard conditions en `when` introducidas en preview (con opt-in) en Kotlin 2.1.0
- https://kotlinlang.org/docs/whatsnew22.html — guard conditions promovidas a estable en Kotlin 2.2.0 (el cheatsheet decía "Kotlin 2.0+", corregido a "Kotlin 2.2+")
- https://kotlinlang.org/docs/releases.html — proceso de release de Kotlin

### Versión de Kotlin y toolchain
- Kotlin 2.3.20 como versión estable actual (ya verificado en la sección Android Jetpack Compose de este mismo documento)
- https://plugins.gradle.org/plugin/com.google.devtools.ksp y búsqueda en Maven Central — KSP pasó a versionado propio (`2.3.10`), ya no usa el prefijo `<kotlin-version>-<ksp-revision>`; compatible con Kotlin 2.2+

### Librerías (versiones en `build.gradle.kts`)
- https://github.com/Kotlin/kotlinx.coroutines/releases — 1.11.0, basado en Kotlin 2.2.20
- https://github.com/Kotlin/kotlinx.serialization/releases — 1.11.0, basado en Kotlin 2.3.20
- https://ktor.io/docs/releases.html y https://ktor.io/docs/whats-new-350.html — Ktor 3.5.0 (15 may. 2026)
- https://arrow-kt.io/community/blog/2026/06/04/arrow-2-2-3/ — Arrow 2.2.3 (4 jun. 2026); confirma que Arrow 2.x requiere Kotlin 2.2.0 como mínimo (el cheatsheet fijaba Arrow 1.2.4, una major desactualizada)
- https://mvnrepository.com/artifact/io.mockk/mockk — MockK 1.14.7
- https://kotest.io/docs/changelog.html y https://github.com/kotest/kotest/releases — Kotest 6.2.3; confirma breaking changes de la serie 6.0 (remoción de `@AutoScan`, requiere JDK 11+ y Kotlin 2.2+) respecto a la 5.9.1 que tenía el cheatsheet
- https://mvnrepository.com/artifact/io.github.oshai/kotlin-logging-jvm — kotlin-logging-jvm 8.0.4
- https://github.com/qos-ch/logback/releases — logback-classic 1.5.38

---

## C#

_Investigado: julio 2026 (cheatsheet nuevo, primera versión — dividido en carpetas por tema en `programming-languages/csharp/`)._

### Versión de .NET / C#
- https://learn.microsoft.com/en-us/dotnet/core/whats-new/dotnet-10/overview — .NET 10 es la versión LTS actual (soporte de 3 años), lanzada junto con C# 14
- https://www.theregister.com/2025/11/12/net_10_c_14_visual/ — confirma el lanzamiento de .NET 10 LTS y Visual Studio 2026 en noviembre 2025
- https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/configure-language-version — C# 14 requiere .NET 10 como mínimo

### C# 14 (novedades usadas en csharp-moderno.md)
- https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-14 — extension members, `field` keyword, null-conditional assignment, implicit span conversions, partial events/constructors, modificadores en lambdas sin tipo explícito
- https://learn.microsoft.com/en-us/dotnet/core/whats-new/dotnet-10/overview — resumen de C# 14 dentro del overview de .NET 10

### C# 12 / 13 (features previas, siguen vigentes)
- https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-12 — primary constructors (cualquier class/struct), collection expressions con spread `..`, inline arrays
- https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-13 — `params` collections (incluye `ReadOnlySpan<T>`), nuevo tipo `System.Threading.Lock`, partial properties/indexers, `\e` escape sequence

### Testing (xUnit v3) — reutilizado del research de ASP.NET Core en este mismo documento
- https://xunit.net/docs/getting-started/v3/migration — `IAsyncLifetime` usa `ValueTask`, requiere `<OutputType>Exe</OutputType>`
- https://www.nuget.org/packages/FluentAssertions — confirma que v8+ requiere licencia comercial (Xceed); se fija la última 7.x libre (Apache 2.0)

---

## Developer Tools (Git / Docker)

_Investigado: julio 2026. A diferencia de `frameworks/`, estos cheatsheets se mantienen como archivo único — no se dividen en carpetas por tema._

### Git
- https://git-scm.com/docs/git-worktree — comandos y caso de uso oficial de `git worktree`
- Documentación de GitHub sobre firma de commits con SSH (`gpg.format ssh`, `user.signingkey`) — soportado desde Git 2.34+

### Docker — versiones de imágenes base
- https://github.com/nodejs/Release — línea LTS activa de Node.js (24.x) vs Current (26.x, LTS desde oct. 2026)
- https://www.postgresql.org/docs/release/ — última versión estable de PostgreSQL (18.4)
- https://www.postgresql.org/developer/roadmap — PostgreSQL 19 todavía en beta
- Documentación oficial de Redis (licencia tri-license SSPL/RSALv2/AGPLv3 desde Redis 8) y de Valkey (fork BSD-3-Clause)
- https://canonical.com/blog/canonical-releases-ubuntu-26-04-lts-resolute-raccoon — Ubuntu 26.04 LTS
- https://docs.spring.io/spring-boot/reference/packaging/container-images/dockerfiles.html — mismo cambio de `jarmode` aplicado también en el ejemplo Java del cheatsheet de Docker

### Docker Compose Watch / Include
- https://docs.docker.com/compose/how-tos/file-watch/ — sintaxis de `develop.watch` (sync, rebuild, sync+restart)
- https://docs.docker.com/reference/cli/docker/compose/watch/ — comando `docker compose watch`

### Docker Scout
- https://docs.docker.com/reference/cli/docker/scout/quickview/
- https://docs.docker.com/reference/cli/docker/scout/cves/
- https://docs.docker.com/reference/cli/docker/scout/compare/ — marcado como experimental por Docker
- https://docs.docker.com/reference/cli/docker/scout/recommendations/
