# Dart — Programación orientada a objetos

---

## 🏛️ Clases

```dart
class Product {
  // Campos de instancia
  final String id;
  String name;
  double price;
  String? category;

  // Campo estático
  static int _count = 0;
  static int get totalCreated => _count;

  // Constructor primario
  Product({
    required this.id,
    required this.name,
    required this.price,
    this.category,
  }) { _count++; }

  // Named parameters privados (Dart 3.12+) — inicializan el campo _privado
  // directamente, sin initializer list. El llamador usa el nombre sin "_".
  // Product({required this._sku}) equivale a
  // Product({required String sku}) : _sku = sku;

  // Constructor nombrado
  Product.empty()
      : id = UniqueKey().toString(),
        name = "",
        price = 0.0;

  // Constructor factory — puede retornar subclase o instancia cacheada
  factory Product.fromJson(Map<String, dynamic> json) => Product(
    id: json['id'] as String,
    name: json['name'] as String,
    price: (json['price'] as num).toDouble(),
    category: json['category'] as String?,
  );

  // Constructor constante
  const Product.fixed({required this.id, required this.name, required this.price})
      : category = null;

  // Getter
  bool get isExpensive => price > 500;
  String get displayPrice => "\$${price.toStringAsFixed(2)}";

  // Setter
  set discountedPrice(double discount) {
    if (discount < 0 || discount > 1) throw ArgumentError("Descuento inválido");
    price = price * (1 - discount);
  }

  // copyWith
  Product copyWith({String? name, double? price, String? category}) => Product(
    id: id,
    name: name ?? this.name,
    price: price ?? this.price,
    category: category ?? this.category,
  );

  Map<String, dynamic> toJson() => {
    'id': id,
    'name': name,
    'price': price,
    if (category != null) 'category': category,
  };

  @override
  String toString() => "Product(id: $id, name: $name, price: $price)";

  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is Product && runtimeType == other.runtimeType && id == other.id;

  @override
  int get hashCode => id.hashCode;
}

// Inicializador de lista y forwarding
class Point {
  final double x, y, magnitude;

  Point(double x, double y)
      : x = x,
        y = y,
        magnitude = sqrt(x * x + y * y);

  Point.origin() : this(0, 0);
  Point.onXAxis(double x) : this(x, 0);
}

// Clase inmutable con const
class Color {
  final int r, g, b;
  const Color(this.r, this.g, this.b);

  static const red   = Color(255, 0, 0);
  static const green = Color(0, 255, 0);
  static const blue  = Color(0, 0, 255);

  Color mix(Color other) => Color(
    (r + other.r) ~/ 2,
    (g + other.g) ~/ 2,
    (b + other.b) ~/ 2,
  );
}
```

---

## 🧬 Herencia, interfaces e implementación

```dart
// extends — herencia simple
abstract class Animal {
  final String name;
  const Animal(this.name);

  void breathe() => print("$name respira");
  void makeSound();
  String describe() => "Animal: $name";
}

class Dog extends Animal {
  final String breed;
  Dog(String name, this.breed) : super(name);

  @override
  void makeSound() => print("$name: ¡Guau!");

  @override
  String describe() => "${super.describe()}, Raza: $breed";
}

// implements — implementar una o varias interfaces
abstract interface class Flyable {
  void fly();
  double get maxAltitude;
}

abstract interface class Swimmable {
  void swim();
  double get maxDepth;
}

class Duck extends Animal implements Flyable, Swimmable {
  Duck(String name) : super(name);

  @override void makeSound() => print("$name: ¡Cuac!");
  @override void fly() => print("$name vuela bajo");
  @override double get maxAltitude => 100.0;
  @override void swim() => print("$name nada");
  @override double get maxDepth => 2.0;
}

// covariant — sobreescribir con tipo más específico
class Box<T extends Animal> {
  T animal;
  Box(this.animal);
  void setAnimal(covariant T newAnimal) { animal = newAnimal; }
}
```

---

## 🧩 Mixins

```dart
mixin Timestamped {
  late final DateTime _createdAt = DateTime.now();
  DateTime _updatedAt = DateTime.now();

  DateTime get createdAt => _createdAt;
  DateTime get updatedAt => _updatedAt;
  void touch() => _updatedAt = DateTime.now();
}

mixin Auditable {
  final List<String> _auditLog = [];
  void audit(String action) =>
      _auditLog.add("${DateTime.now()}: $action");
  List<String> get auditLog => List.unmodifiable(_auditLog);
}

mixin Validatable {
  bool validate();
  void validateOrThrow() {
    if (!validate()) throw StateError("Validation failed for $runtimeType");
  }
}

// on — restringir a qué clase se puede aplicar el mixin
mixin Logger on Service {
  void log(String msg) => print("[$runtimeType] $msg");
}

// Aplicar múltiples mixins
class Order extends BaseEntity with Timestamped, Auditable, Validatable {
  String id;
  List<OrderItem> items;
  Order(this.id, this.items);

  @override
  bool validate() => items.isNotEmpty && id.isNotEmpty;
}
```

---

## 🔌 Extensions

```dart
extension StringExtensions on String {
  String get capitalized =>
      isEmpty ? this : "${this[0].toUpperCase()}${substring(1)}";

  bool get isValidEmail =>
      RegExp(r'^[\w\.-]+@[\w\.-]+\.\w{2,}$').hasMatch(this);

  bool get isNumeric => double.tryParse(this) != null;

  String truncate(int maxLength, {String ellipsis = "..."}) =>
      length <= maxLength ? this : "${substring(0, maxLength)}$ellipsis";

  String toSnakeCase() => replaceAllMapped(
    RegExp(r'(?<=[a-z])[A-Z]'),
    (m) => '_${m.group(0)!.toLowerCase()}',
  ).toLowerCase();
}

extension IntExtensions on int {
  Duration get seconds => Duration(seconds: this);
  Duration get milliseconds => Duration(milliseconds: this);
  Duration get minutes => Duration(minutes: this);
  List<int> get range => List.generate(this, (i) => i);
  void times(void Function(int) fn) {
    for (int i = 0; i < this; i++) fn(i);
  }
}

extension DateTimeExtensions on DateTime {
  bool get isToday {
    final now = DateTime.now();
    return day == now.day && month == now.month && year == now.year;
  }
  bool get isPast => isBefore(DateTime.now());
  bool get isFuture => isAfter(DateTime.now());

  String get timeAgo {
    final diff = DateTime.now().difference(this);
    if (diff.inDays > 365) return "hace ${diff.inDays ~/ 365} año(s)";
    if (diff.inDays > 30) return "hace ${diff.inDays ~/ 30} mes(es)";
    if (diff.inDays > 0) return "hace ${diff.inDays}d";
    if (diff.inHours > 0) return "hace ${diff.inHours}h";
    if (diff.inMinutes > 0) return "hace ${diff.inMinutes}min";
    return "ahora mismo";
  }

  DateTime get startOfDay => DateTime(year, month, day);
  DateTime get endOfDay => DateTime(year, month, day, 23, 59, 59, 999);
}

extension ListExtensions<T> on List<T> {
  T? get firstOrNull => isEmpty ? null : first;
  T? get lastOrNull => isEmpty ? null : last;

  T? firstWhereOrNull(bool Function(T) test) {
    for (final e in this) if (test(e)) return e;
    return null;
  }

  List<List<T>> chunk(int size) => [
    for (int i = 0; i < length; i += size)
      sublist(i, (i + size).clamp(0, length))
  ];

  Map<K, List<T>> groupBy<K>(K Function(T) keyFn) {
    final map = <K, List<T>>{};
    for (final e in this) map.putIfAbsent(keyFn(e), () => []).add(e);
    return map;
  }

  List<T> distinctBy<K>(K Function(T) keyFn) {
    final seen = <K>{};
    return where((e) => seen.add(keyFn(e))).toList();
  }
}

extension MapExtensions<K, V> on Map<K, V> {
  Map<K, V> whereEntries(bool Function(K key, V value) test) =>
      Map.fromEntries(entries.where((e) => test(e.key, e.value)));
}

// Uso
"hola mundo".capitalized;           // "Hola mundo"
"test@email.com".isValidEmail;      // true
5.seconds;                           // Duration(seconds: 5)
5.times((i) => print(i));
[1, 2, 3, 4, 5].chunk(2);          // [[1,2],[3,4],[5]]
DateTime.now().isToday;             // true
```

---

## 🎭 Enums avanzados

```dart
// Enum básico
enum Direction { north, south, east, west }

// Enum con valores y métodos (Dart 2.17+)
enum Status {
  pending("Pendiente"),
  active("Activo"),
  inactive("Inactivo"),
  suspended("Suspendido");

  const Status(this.label);
  final String label;

  bool get isActive => this == Status.active;
  Status toggle() => this == active ? inactive : active;

  static Status fromString(String value) => Status.values.firstWhere(
    (s) => s.name == value,
    orElse: () => throw ArgumentError("Status inválido: $value"),
  );
}

// Enum con interface
enum Planet implements Comparable<Planet> {
  mercury(3.303e+23, 2.4397e6),
  venus(4.869e+24, 6.0518e6),
  earth(5.976e+24, 6.37814e6),
  mars(6.421e+23, 3.3972e6);

  const Planet(this.mass, this.radius);
  final double mass;
  final double radius;

  static const double G = 6.67430e-11;
  double get surfaceGravity => G * mass / (radius * radius);
  double weightOn(double weight) =>
      weight * surfaceGravity / Planet.earth.surfaceGravity;

  @override
  int compareTo(Planet other) => mass.compareTo(other.mass);
}

// Propiedades built-in
Status.active.name;                    // "active"
Status.active.index;                   // 1
Status.values;                         // [pending, active, ...]
Status.values.byName("active");        // Status.active

// Switch exhaustivo con enum (sin _ default)
String describe(Direction dir) => switch (dir) {
  Direction.north => "Norte",
  Direction.south => "Sur",
  Direction.east  => "Este",
  Direction.west  => "Oeste",
};
```

---

## 🧮 Generics

```dart
// Clase genérica
class Stack<T> {
  final List<T> _items = [];

  void push(T item) => _items.add(item);

  T pop() {
    if (isEmpty) throw StateError("Stack vacío");
    return _items.removeLast();
  }

  T get peek => isEmpty ? throw StateError("Stack vacío") : _items.last;
  bool get isEmpty => _items.isEmpty;
  int get size => _items.length;
}

// Bounds — restringir el tipo genérico
class NumberBox<T extends num> {
  T value;
  NumberBox(this.value);
  T get doubled => (value * 2) as T;
  bool get isPositive => value > 0;
}

// Funciones genéricas
T identity<T>(T value) => value;
T? safeCast<T>(Object? value) => value is T ? value : null;
List<T> repeat<T>(T value, int times) => List.generate(times, (_) => value);

// Result type genérico
sealed class Result<T> {
  const Result();
  factory Result.success(T data) = Success<T>;
  factory Result.failure(String message, {Object? cause}) = Failure<T>;

  T getOrElse(T defaultValue);
  T? getOrNull();
  Result<R> map<R>(R Function(T) transform);

  void when({
    required void Function(T data) success,
    required void Function(String message, Object? cause) failure,
  });
}

class Success<T> extends Result<T> {
  final T data;
  const Success(this.data);

  @override T getOrElse(T _) => data;
  @override T? getOrNull() => data;
  @override Result<R> map<R>(R Function(T) transform) => Success(transform(data));
  @override void when({required success, required failure}) => success(data);
}

class Failure<T> extends Result<T> {
  final String message;
  final Object? cause;
  const Failure(this.message, {this.cause});

  @override T getOrElse(T defaultValue) => defaultValue;
  @override T? getOrNull() => null;
  @override Result<R> map<R>(R Function(T) _) => Failure<R>(message, cause: cause);
  @override void when({required success, required failure}) =>
      failure(message, cause);
}

// Uso
final ok = Result.success(42);
ok.map((n) => n * 2);          // Success(84)
ok.getOrElse(0);               // 42

final err = Result<int>.failure("no encontrado");
err.map((n) => n * 2);         // Failure("no encontrado")
err.getOrElse(0);              // 0
```

---

## 📝 Typedefs

```dart
// Función
typedef JsonMap = Map<String, dynamic>;
typedef Predicate<T> = bool Function(T value);
typedef Transformer<A, B> = B Function(A input);
typedef VoidCallback = void Function();
typedef AsyncCallback = Future<void> Function();
typedef ErrorHandler = void Function(Object error, StackTrace? stack);

// Tipos complejos
typedef StringList = List<String>;
typedef ProductMap = Map<String, Product>;

// Uso
Predicate<int> isPositive = (n) => n > 0;
Predicate<String> isNotEmpty = (s) => s.isNotEmpty;
Transformer<String, int> stringLength = (s) => s.length;

void processItems<T>(
  List<T> items,
  void Function(T item, int index) processor,
) {
  for (int i = 0; i < items.length; i++) processor(items[i], i);
}
```
