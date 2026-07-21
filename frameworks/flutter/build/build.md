# Flutter — Build y distribución

---

## 🏗️ Flavors y build

```yaml
# pubspec.yaml — dart-define-from-file (Flutter 3.7+)
# Crear archivos de configuración:
# config/dev.json, config/staging.json, config/prod.json
```

```json
// config/dev.json
{
  "FLAVOR": "dev",
  "API_BASE_URL": "https://dev.api.miapp.com/v1",
  "FIREBASE_PROJECT_ID": "miapp-dev",
  "ENABLE_LOGGING": "true",
  "SENTRY_DSN": ""
}

// config/prod.json
{
  "FLAVOR": "prod",
  "API_BASE_URL": "https://api.miapp.com/v1",
  "FIREBASE_PROJECT_ID": "miapp-prod",
  "ENABLE_LOGGING": "false",
  "SENTRY_DSN": "https://key@sentry.io/project"
}
```

```dart
// config/app_config.dart
class AppConfig {
  static const flavor = String.fromEnvironment('FLAVOR', defaultValue: 'dev');
  static const apiBaseUrl = String.fromEnvironment('API_BASE_URL');
  static const enableLogging = bool.fromEnvironment('ENABLE_LOGGING', defaultValue: true);
  static const sentryDsn = String.fromEnvironment('SENTRY_DSN');

  static bool get isDev => flavor == 'dev';
  static bool get isStaging => flavor == 'staging';
  static bool get isProd => flavor == 'prod';
}

// Usar en la app
final dio = Dio(BaseOptions(baseUrl: AppConfig.apiBaseUrl));
if (AppConfig.enableLogging) dio.interceptors.add(PrettyDioLogger());
```

```bash
# Comandos de build con flavors
flutter run --dart-define-from-file=config/dev.json
flutter run --dart-define-from-file=config/staging.json
flutter build apk --dart-define-from-file=config/prod.json --release
flutter build appbundle --dart-define-from-file=config/prod.json --release
flutter build ipa --dart-define-from-file=config/prod.json --release

# Con ofuscación (recomendado para producción)
flutter build apk \
  --dart-define-from-file=config/prod.json \
  --release \
  --obfuscate \
  --split-debug-info=build/debug-info/android

flutter build ipa \
  --dart-define-from-file=config/prod.json \
  --release \
  --obfuscate \
  --split-debug-info=build/debug-info/ios

# Split por ABI (Android — APKs más pequeños)
flutter build apk --split-per-abi --release

# Lanzador de VS Code / Android Studio
# .vscode/launch.json
{
  "configurations": [
    {
      "name": "Dev",
      "request": "launch",
      "type": "dart",
      "args": ["--dart-define-from-file=config/dev.json"]
    },
    {
      "name": "Prod",
      "request": "launch",
      "type": "dart",
      "args": ["--dart-define-from-file=config/prod.json"]
    }
  ]
}
```


