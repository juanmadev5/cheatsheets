# Dart — Herramientas — CLI, pubspec.yaml y buenas prácticas

---

## 🛠️ Dart CLI

```bash
# Ejecutar
dart run lib/main.dart
dart run bin/server.dart
dart run                           # Usa main de pubspec

# Compilar
dart compile exe bin/main.dart            # Binario nativo standalone
dart compile exe bin/main.dart -o app     # Con nombre de salida
dart compile js lib/main.dart             # JavaScript
dart compile aot-snapshot lib/main.dart   # AOT (más rápido)
dart compile jit-snapshot lib/main.dart   # JIT con calentamiento

# Paquetes
dart pub get                         # Instalar dependencias
dart pub upgrade                     # Actualizar compatibles
dart pub upgrade --major-versions    # Actualizar mayores
dart pub outdated                    # Ver desactualizados
dart pub add dio                     # Agregar paquete
dart pub add dev:build_runner        # Agregar dev dependency
dart pub remove dio
dart pub publish --dry-run           # Verificar sin publicar
dart pub deps                        # Árbol de dependencias

# Code generation
dart run build_runner build
dart run build_runner build --delete-conflicting-outputs
dart run build_runner watch          # Modo watch
dart run build_runner clean

# Análisis y formato
dart analyze
dart analyze lib/
dart format .
dart format lib/main.dart
dart format --set-exit-if-changed .  # Para CI

# Fixes automáticos
dart fix --apply
dart fix --dry-run

# Tests
dart test
dart test test/unit/
dart test test/my_test.dart
dart test --name "nombre del test"
dart test --tags unit
dart test --coverage=coverage

# Cobertura
dart pub global activate coverage
dart run test --coverage=coverage/
dart pub global run coverage:format_coverage \
    --packages=.dart_tool/package_config.json \
    --report-on=lib --lcov \
    -i coverage -o coverage/lcov.info

# Info
dart info
dart --version
dart pub global activate <tool>
dart pub global run <tool>
```

---

## 📄 pubspec.yaml

```yaml
name: mi_app
description: Descripción de mi aplicación.
version: 1.2.3      # semver: major.minor.patch
publish_to: "none"  # Evitar publicación accidental

environment:
  sdk: ">=3.3.0 <4.0.0"

dependencies:
  # Versión compatible (^1.2.3 = >=1.2.3 <2.0.0)
  dio: ^5.7.0
  riverpod: ^2.6.0
  freezed_annotation: ^2.4.4
  json_annotation: ^4.9.0

  # Versión exacta
  some_package: 1.0.0

  # Desde git
  my_private_package:
    git:
      url: https://github.com/usuario/repo.git
      ref: main    # rama, tag o commit hash

  # Desde path local (monorepos, desarrollo)
  shared_utils:
    path: ../shared_utils

  # Registry privado
  enterprise_package:
    hosted:
      name: enterprise_package
      url: https://my-private-pub.company.com
    version: ^1.0.0

dev_dependencies:
  freezed: ^2.5.7
  json_serializable: ^6.8.0
  build_runner: ^2.4.13
  test: ^1.25.0
  mocktail: ^1.0.4
  lints: ^5.0.0
  very_good_analysis: ^7.0.0   # Lint más estricto

# Sobreescribir versiones transitivas
dependency_overrides:
  some_transitive_dep: ^2.0.0

# Flutter específico
flutter:
  uses-material-design: true

  assets:
    - assets/images/
    - assets/fonts/
    - assets/config.json

  fonts:
    - family: Roboto
      fonts:
        - asset: assets/fonts/Roboto-Regular.ttf
        - asset: assets/fonts/Roboto-Bold.ttf
          weight: 700
        - asset: assets/fonts/Roboto-Italic.ttf
          style: italic
```

```yaml
# analysis_options.yaml
include: package:lints/recommended.yaml
# o más estricto:
# include: package:very_good_analysis/analysis_options.yaml

analyzer:
  exclude:
    - "**/*.g.dart"
    - "**/*.freezed.dart"
  errors:
    invalid_annotation_target: ignore
    missing_required_param: error
    unused_import: warning

linter:
  rules:
    prefer_const_constructors: true
    prefer_const_declarations: true
    avoid_print: true
    prefer_final_locals: true
    prefer_final_fields: true
    use_super_parameters: true
    require_trailing_commas: true
```

---

## 💡 Buenas prácticas y estilo

### Nomenclatura

```dart
// UpperCamelCase — clases, enums, typedefs, type parameters
class UserProfile {}
enum OrderStatus { pending, active }
typedef JsonMap = Map<String, dynamic>;
class Repository<T> {}

// lowerCamelCase — variables, parámetros, funciones, getters/setters
final userName = "Juan";
void fetchUser() {}
bool get isActive => _isActive;

// snake_case — archivos y carpetas
// user_profile.dart, product_repository.dart, order_status.dart

// SCREAMING_SNAKE_CASE — constantes a nivel de librería (no en clase)
const double maxRetries = 3;
const String defaultLocale = "es";

// Prefijo _ — privado al archivo/librería
int _counter = 0;
void _privateMethod() {}
class _PrivateHelper {}
```

---

### Reglas clave

```dart
// ✅ Preferir const
const Color myColor = Color(0xFF2563EB);
const EdgeInsets padding = EdgeInsets.all(16);

// ✅ Preferir final sobre var
final name = computeName();

// ✅ Usar => para funciones de una línea
int square(int x) => x * x;

// ✅ Nombrar parámetros cuando hay más de 2
createUser(name: "Juan", email: "j@j.com", age: 28);

// ✅ is en lugar de runtimeType ==
if (obj is String) {}

// ✅ ?. en lugar de null check + acceso
user?.name;

// ✅ Collection literals
<String>[];
<String, int>{};

// ✅ if con spread para listas condicionales
final items = ["base", if (isPro) "pro-feature"];

// ✅ Trailing commas — mejora el formato automático
Widget build(BuildContext context) => Column(
  children: [
    Text("hola"),
    Text("mundo"),  // Trailing comma
  ],
);

// ✅ Documentar API pública con ///
/// Fetches a user by their unique [id].
///
/// Throws [UserNotFoundException] if no user is found.
/// Returns `null` if [id] is empty.
Future<User?> fetchUser(String id) async { ... }
```

---

### Patrones de diseño en Dart

```dart
// Singleton
class AppConfig {
  static final AppConfig _instance = AppConfig._internal();
  factory AppConfig() => _instance;
  AppConfig._internal();
  String apiUrl = "https://api.example.com";
}

// Observer con Stream
class EventBus {
  static final EventBus instance = EventBus._();
  EventBus._();

  final _controllers = <Type, StreamController>{};

  Stream<T> on<T>() {
    _controllers[T] ??= StreamController<T>.broadcast();
    return (_controllers[T] as StreamController<T>).stream;
  }

  void emit<T>(T event) {
    (_controllers[T] as StreamController<T>?)?.add(event);
  }

  void dispose() {
    for (final c in _controllers.values) c.close();
    _controllers.clear();
  }
}

// Uso
EventBus.instance.on<UserLoggedIn>().listen((e) => print(e.userId));
EventBus.instance.emit(UserLoggedIn(userId: "123"));

// Builder pattern
class QueryBuilder {
  String _table = "";
  final List<String> _conditions = [];
  final List<String> _columns = [];
  int? _limit;
  String? _orderBy;

  QueryBuilder from(String table) => this.._table = table;
  QueryBuilder select(List<String> cols) => this.._columns.addAll(cols);
  QueryBuilder where(String cond) => this.._conditions.add(cond);
  QueryBuilder limit(int n) => this.._limit = n;
  QueryBuilder orderBy(String col) => this.._orderBy = col;

  String build() {
    final cols = _columns.isEmpty ? "*" : _columns.join(", ");
    var q = "SELECT $cols FROM $_table";
    if (_conditions.isNotEmpty) q += " WHERE ${_conditions.join(' AND ')}";
    if (_orderBy != null) q += " ORDER BY $_orderBy";
    if (_limit != null) q += " LIMIT $_limit";
    return q;
  }
}

final sql = QueryBuilder()
    .from("users")
    .select(["id", "name", "email"])
    .where("active = true")
    .where("age > 18")
    .orderBy("name")
    .limit(10)
    .build();

// Memoización
Map<A, R> Function(A) memoize<A, R>(R Function(A) fn) {
  final cache = <A, R>{};
  return (A arg) => cache.putIfAbsent(arg, () => fn(arg));
}

final cachedFactorial = memoize(factorial);
cachedFactorial(10);   // Calcula y cachea
cachedFactorial(10);   // Retorna del cache

// Retry con backoff exponencial
Future<T> withRetry<T>(
  Future<T> Function() operation, {
  int maxAttempts = 3,
  Duration initialDelay = const Duration(seconds: 1),
}) async {
  int attempt = 0;
  while (true) {
    try {
      return await operation();
    } catch (e) {
      attempt++;
      if (attempt >= maxAttempts) rethrow;
      final delay = initialDelay * (1 << (attempt - 1));  // 1s, 2s, 4s
      await Future.delayed(delay);
    }
  }
}
