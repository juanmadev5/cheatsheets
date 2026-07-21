# Flutter — Dart moderno (Dart 3.x)

---

## 🎯 Dart moderno — Dart 3.x

### Null safety

```dart
// Tipos nullable vs non-nullable
String name = "Juan";          // Non-nullable — nunca puede ser null
String? nickname = null;       // Nullable — puede ser null

// Operadores null safety
String display = nickname ?? "Sin apodo";        // ?? — valor por defecto si null
int? length = nickname?.length;                  // ?. — acceso seguro (null si es null)
String forced = nickname!;                       // ! — aserción (lanza si es null, evitar)
nickname ??= "Default";                          // ??= — asignar solo si es null

// late — inicialización diferida (no null pero se asigna después)
late String userId;             // Se asignará antes de usarse
late final String token;        // Asignación única, luego inmutable

// required — parámetros nombrados obligatorios
class UserCard extends StatelessWidget {
  const UserCard({
    super.key,
    required this.name,         // Obligatorio
    this.avatarUrl,             // Opcional (nullable)
    this.onTap,                 // Opcional
  });

  final String name;
  final String? avatarUrl;
  final VoidCallback? onTap;
}
```

---

### Records y destructuring (Dart 3+)

```dart
// Records — tuplas con tipo estático
(String, int) getUserInfo() => ("Juan", 28);

final (name, age) = getUserInfo();   // Destructuring
print("$name tiene $age años");

// Records con campos nombrados
({String name, int age, String email}) getUser() =>
    (name: "Juan", age: 28, email: "juan@email.com");

final user = getUser();
print(user.name);    // Acceso por nombre
print(user.age);

// Destructuring en múltiples variables
final (:name, :age, :email) = getUser();

// En listas y mapas
final [first, second, ...rest] = [1, 2, 3, 4, 5];
final {"key1": v1, "key2": v2} = {"key1": "a", "key2": "b"};
```

---

### Pattern matching y sealed classes (Dart 3+)

```dart
// sealed class — jerarquía cerrada, el compilador sabe todos los subtipos
sealed class AuthState {}
class Unauthenticated extends AuthState {}
class Authenticated extends AuthState {
  final String userId;
  final String email;
  Authenticated({required this.userId, required this.email});
}
class AuthLoading extends AuthState {}
class AuthError extends AuthState {
  final String message;
  AuthError(this.message);
}

// switch exhaustivo — el compilador exige todos los casos
String describeAuth(AuthState state) => switch (state) {
  Unauthenticated() => "No autenticado",
  Authenticated(:final email) => "Autenticado como $email",
  AuthLoading() => "Cargando...",
  AuthError(:final message) => "Error: $message",
};

// Pattern matching con if-case
void handleState(AuthState state) {
  if (state case Authenticated(:final userId, :final email)) {
    print("User $userId: $email");
  }
}

// Switch con guards
String classify(int n) => switch (n) {
  0 => "cero",
  int x when x < 0 => "negativo",
  int x when x.isEven => "positivo par",
  _ => "positivo impar",
};

// sealed class para resultados (reemplaza Either)
sealed class Result<T> {}
class Success<T> extends Result<T> {
  final T data;
  Success(this.data);
}
class Failure<T> extends Result<T> {
  final String message;
  final Object? error;
  Failure(this.message, {this.error});
}

// Uso en repositorio
Future<Result<List<Product>>> fetchProducts() async {
  try {
    final data = await api.getProducts();
    return Success(data.map((e) => e.toDomain()).toList());
  } on DioException catch (e) {
    return Failure("Error de red: ${e.message}");
  } catch (e) {
    return Failure("Error inesperado", error: e);
  }
}

// Consumo exhaustivo
switch (await fetchProducts()) {
  case Success(:final data): showProducts(data);
  case Failure(:final message): showError(message);
}
```

---

### Extensions, Mixins y Generics

```dart
// Extension methods — agregar funcionalidad a tipos existentes
extension StringExtensions on String {
  String capitalize() =>
      isEmpty ? this : "${this[0].toUpperCase()}${substring(1)}";

  bool get isValidEmail =>
      RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(this);

  String truncate(int maxLength, {String ellipsis = "..."}) =>
      length <= maxLength ? this : "${substring(0, maxLength)}$ellipsis";
}

extension DateTimeExtensions on DateTime {
  bool get isToday {
    final now = DateTime.now();
    return day == now.day && month == now.month && year == now.year;
  }
  String get timeAgo {
    final diff = DateTime.now().difference(this);
    if (diff.inDays > 0) return "hace ${diff.inDays}d";
    if (diff.inHours > 0) return "hace ${diff.inHours}h";
    return "hace ${diff.inMinutes}m";
  }
}

extension ListExtensions<T> on List<T> {
  List<T> get nonNulls => whereType<T>().toList();
  T? firstWhereOrNull(bool Function(T) test) {
    for (final e in this) if (test(e)) return e;
    return null;
  }
}

// Uso
"hola mundo".capitalize();     // "Hola mundo"
"test@email.com".isValidEmail; // true
DateTime.now().isToday;        // true

// Mixins — reutilizar comportamiento sin herencia
mixin LoadingMixin {
  bool _isLoading = false;
  bool get isLoading => _isLoading;

  void setLoading(bool value) {
    _isLoading = value;
  }
}

mixin ValidationMixin {
  String? validateEmail(String? value) {
    if (value == null || value.isEmpty) return "Campo requerido";
    if (!value.isValidEmail) return "Email inválido";
    return null;
  }
  String? validatePassword(String? value) {
    if (value == null || value.length < 8) return "Mínimo 8 caracteres";
    return null;
  }
}

class AuthViewModel extends ChangeNotifier with LoadingMixin, ValidationMixin {
  Future<void> login(String email, String password) async {
    setLoading(true);
    notifyListeners();
    // ...
    setLoading(false);
    notifyListeners();
  }
}

// Generics
class Repository<T> {
  final Map<String, T> _cache = {};

  T? getFromCache(String key) => _cache[key];
  void saveToCache(String key, T value) => _cache[key] = value;

  Future<T> fetchById(String id, Future<T> Function(String) fetcher) async {
    final cached = getFromCache(id);
    if (cached != null) return cached;
    final result = await fetcher(id);
    saveToCache(id, result);
    return result;
  }
}

// typedef — alias de tipos
typedef JsonMap = Map<String, dynamic>;
typedef VoidAsyncCallback = Future<void> Function();
typedef ProductMapper = Product Function(JsonMap json);
```

---

### Modelos con freezed + json_serializable

```yaml
# pubspec.yaml
dependencies:
  freezed_annotation: ^3.1.0
  json_annotation: ^4.12.0

dev_dependencies:
  freezed: ^3.2.5
  json_serializable: ^6.14.0
  build_runner: ^2.15.2
```

```dart
// product.dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'product.freezed.dart';
part 'product.g.dart';

@freezed
abstract class Product with _$Product {
  const factory Product({
    required String id,
    required String name,
    required double price,
    @JsonKey(name: 'image_url') required String imageUrl,
    String? category,
    @Default([]) List<String> tags,
    @Default(false) bool isFavorite,
    DateTime? createdAt,
  }) = _Product;

  factory Product.fromJson(JsonMap json) => _$ProductFromJson(json);
}

// Uso — copyWith, ==, hashCode, toString generados automáticamente
final product = Product(id: "1", name: "Laptop", price: 999.99, imageUrl: "...");
final updated = product.copyWith(price: 899.99, isFavorite: true);
final json = product.toJson();
final fromJson = Product.fromJson(json);

// Union types con freezed (equivalente a sealed class)
@freezed
sealed class AuthState with _$AuthState {
  const factory AuthState.initial() = AuthInitial;
  const factory AuthState.loading() = AuthLoading;
  const factory AuthState.authenticated({required User user}) = AuthAuthenticated;
  const factory AuthState.unauthenticated() = AuthUnauthenticated;
  const factory AuthState.error({required String message}) = AuthError;
}

// Uso con map/when
state.when(
  initial: () => const SplashScreen(),
  loading: () => const LoadingIndicator(),
  authenticated: (user) => HomeScreen(user: user),
  unauthenticated: () => const LoginScreen(),
  error: (message) => ErrorScreen(message: message),
);

// maybeWhen — solo manejar algunos casos
state.maybeWhen(
  authenticated: (user) => Text(user.name),
  orElse: () => const SizedBox.shrink(),
);

// Generar código
// dart run build_runner build --delete-conflicting-outputs
// dart run build_runner watch  (modo watch durante desarrollo)
```


