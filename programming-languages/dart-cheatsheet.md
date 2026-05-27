# Dart — Cheatsheet

---

## 📑 Índice

### Base
- [Tipos de datos y variables](#-tipos-de-datos-y-variables)
- [Operadores](#-operadores)
- [Control de flujo — if, for, while, switch](#-control-de-flujo)
- [Funciones](#-funciones)

### Null Safety
- [Null safety — ?, ?., ??, ??=, !, late, required](#️-null-safety)

### Dart 3 — Características modernas
- [Records y destructuring](#-records-y-destructuring)
- [Pattern matching — switch, if-case, guards, OR patterns](#-pattern-matching)
- [Sealed classes y class modifiers](#-sealed-classes-y-class-modifiers)

### Programación orientada a objetos
- [Clases — constructor, factory, named constructor, getter, setter](#️-clases)
- [Herencia, interfaces e implementación](#-herencia-interfaces-e-implementación)
- [Mixins](#-mixins)
- [Extensions](#-extensions)
- [Enums avanzados](#-enums-avanzados)
- [Generics — bounds, Result type](#-generics)
- [Typedefs](#-typedefs)

### Colecciones
- [Listas](#-listas)
- [Mapas](#️-mapas)
- [Sets](#-sets)
- [Operaciones funcionales — map, filter, reduce, fold, expand](#-operaciones-funcionales)
- [Spread, collection if, collection for](#-spread-collection-if-collection-for)

### Async
- [Future — async/await, wait, any, timeout, Completer](#-future--asyncawait)
- [Stream — listen, await for, StreamController, broadcast, transform](#-stream)
- [Isolates — Isolate.run, Isolate.spawn, comunicación bidireccional](#-isolates)

### Manejo de errores
- [Excepciones — throw, try/catch/on/finally, custom exceptions, assert](#️-excepciones)

### Sistema de tipos y patrones avanzados
- [Type checks — is, as, is!, promotion, dynamic, callable classes](#-sistema-de-tipos-avanzado)
- [Funciones de primera clase y closures](#-funciones-de-primera-clase-y-closures)

### Dart Core
- [String — métodos esenciales, StringBuffer](#-string--métodos-esenciales)
- [DateTime y Duration](#-datetime-y-duration)
- [Matemáticas — dart:math](#-matemáticas--dartmath)
- [Expresiones regulares — RegExp](#-expresiones-regulares--regexp)

### Entrada/Salida
- [dart:io — archivos, directorios, stdin/stdout, Platform, Process](#-dartio--archivos-y-directorios)
- [dart:convert — JSON, UTF-8, Base64, HTML escape, codecs](#-dartconvert--json-y-codificación)

### Herramientas
- [dart CLI — run, compile, test, pub, fix, analyze, coverage](#️-dart-cli)
- [pubspec.yaml — estructura completa, git/path deps, overrides](#-pubspecyaml)
- [Buenas prácticas — estilo, nomenclatura, patrones de diseño](#-buenas-prácticas-y-estilo)

---

## 📊 Tipos de datos y variables

### Variables y constantes

```dart
// var — inferencia de tipo, mutable
var name = "Juan";
var age = 28;
var price = 99.99;

// Tipo explícito
String country = "Paraguay";
int count = 0;
double ratio = 3.14;
bool isActive = true;

// final — asignación única en runtime
final createdAt = DateTime.now();
final List<String> roles = ["admin", "user"];

// const — constante en tiempo de compilación
const pi = 3.14159;
const maxRetries = 3;
const appName = "MiApp";
const colors = ["red", "green", "blue"];  // Lista constante e inmutable

// late — inicialización diferida, garantiza asignación antes del uso
late String userId;
late final String token;     // late final: asignación única pero diferida

// dynamic — tipo dinámico (evitar en lo posible)
dynamic anything = 42;
anything = "ahora soy string";   // Válido pero no type-safe
```

---

### Tipos primitivos

```dart
// int — enteros (64-bit en VM, 32-bit en web)
int a = 42;
int hex = 0xFF;
int binary = 0b1010;
int big = 1_000_000;         // Separador de miles (Dart 3.x)
int.parse("42");
int.tryParse("abc");         // null si falla

// double — punto flotante (64-bit IEEE 754)
double b = 3.14;
double scientific = 1.5e3;   // 1500.0
double.parse("3.14");
double.infinity;
double.nan;

// num — supertype de int y double
num value = 42;
value = 3.14;

// String — UTF-16
String s = "hola";
String raw = r"no\nescapa";           // Raw string
String multiline = """
  Línea 1
  Línea 2
""";
String interp = "Hola $name, tenés ${age + 1} años";

// bool
bool t = true;
bool f = false;

// Symbol
Symbol sym = #myMethod;
```

---

### Conversiones de tipos

```dart
// int ↔ double
double d = 42.toDouble();
int back = d.toInt();          // Trunca
int rounded = d.round();
int floor = d.floor();
int ceil = d.ceil();

// String ↔ números
String s = 42.toString();
String formatted = 3.14159.toStringAsFixed(2);   // "3.14"
String exp = 12345.toStringAsExponential(2);      // "1.23e+4"
int parsed = int.parse("42");
double parsedD = double.parse("3.14");
int? safe = int.tryParse("abc");                  // null

// String ↔ List<int> (code units)
List<int> codes = "Hola".codeUnits;
String fromCodes = String.fromCharCodes(codes);
```

---

## ➕ Operadores

```dart
// Aritméticos
a + b; a - b; a * b;
a / b;    // División (siempre double)
a ~/ b;   // División entera (int)
a % b;    // Módulo
-a;       // Negación unaria

// Relacionales
a == b; a != b; a > b; a < b; a >= b; a <= b;

// Lógicos
a && b; a || b; !a;

// Bitwise (sobre enteros)
a & b; a | b; a ^ b; ~a; a << 2; a >> 2; a >>> 2;

// Asignación compuesta
a += b; a -= b; a *= b; a /= b; a ~/= b; a %= b;
a &= b; a |= b; a ^= b; a <<= 2; a >>= 2;

// Null-aware
a ?? b;          // b si a es null
a ??= b;         // Asignar b a a solo si a es null
a?.method();     // Llamar solo si a no es null
a?[key];         // Index solo si a no es null

// Cascade — encadenar sobre el mismo objeto
final paint = Paint()
  ..color = Colors.blue
  ..strokeWidth = 2.0
  ..style = PaintingStyle.fill;

// Null cascade
object?..method()..otherMethod();

// Spread en colecciones
final list = [...listA, ...listB];
final map = {...mapA, ...mapB};

// Condicional ternario
final label = isActive ? "Activo" : "Inactivo";

// Type test
obj is String;     // true si obj es String
obj is! String;    // true si NO es String
obj as String;     // Cast — lanza si falla

// Igualdad de identidad
identical(a, b);   // true si son el mismo objeto en memoria
```

---

## 🔀 Control de flujo

### if / else

```dart
if (score >= 90) {
  print("Excelente");
} else if (score >= 70) {
  print("Bueno");
} else {
  print("Necesita mejorar");
}

// if como expresión (Dart 3.x)
final label = if (score >= 90) "Excelente" else "Mejorable";

// if con pattern (Dart 3.x)
if (response case {'status': 200, 'data': final data}) {
  process(data);
}
```

---

### Bucles

```dart
// for clásico
for (int i = 0; i < 10; i++) { print(i); }

// for-in — cualquier Iterable
for (final item in items) { print(item); }

// forEach
items.forEach((item) => print(item));

// while
int i = 0;
while (i < 10) { print(i++); }

// do-while — ejecuta al menos una vez
do { print("ejecutado"); } while (condition);

// break y continue
for (final n in numbers) {
  if (n == 0) continue;
  if (n < 0) break;
  print(1 / n);
}

// Labels — break/continue en bucles anidados
outer:
for (int i = 0; i < 3; i++) {
  for (int j = 0; j < 3; j++) {
    if (j == 1) continue outer;
    if (i == 2) break outer;
  }
}
```

---

### Switch

```dart
// Switch como expresión (Dart 3.x) — retorna valor, exhaustivo
final result = switch (status) {
  200 => "OK",
  404 => "Not Found",
  500 => "Server Error",
  _   => "Unknown $status",
};

// Switch statement con patterns
switch (shape) {
  case Circle(radius: final r):
    print("Área: ${pi * r * r}");
  case Rectangle(width: final w, height: final h):
    print("Área: ${w * h}");
  case _:
    print("Forma desconocida");
}

// Switch con guards
switch (score) {
  case int n when n >= 90: print("Excelente");
  case int n when n >= 70: print("Bueno");
  default: print("Insuficiente");
}

// OR patterns — múltiples valores por caso
switch (day) {
  case "Saturday" || "Sunday": print("Fin de semana");
  default: print("Día laboral");
}
```

---

## 🔧 Funciones

```dart
// Arrow function (una expresión)
int add(int a, int b) => a + b;

// Función con cuerpo
int multiply(int a, int b) {
  return a * b;
}

// Void
void greet(String name) => print("Hola, $name!");

// Parámetros posicionales opcionales [con default]
String repeat(String s, [int times = 2]) => s * times;
repeat("ho");       // "hoho"
repeat("ho", 3);    // "hohoho"

// Parámetros nombrados {required y opcionales}
void createUser({
  required String name,
  required String email,
  int age = 0,
  String? role,
}) => print("$name - $email");

createUser(name: "Juan", email: "juan@email.com", age: 28);

// Funciones como parámetros (higher-order)
List<T> transform<T>(List<T> list, T Function(T) fn) =>
    list.map(fn).toList();

transform([1, 2, 3], (x) => x * 2);   // [2, 4, 6]

// Función que retorna función
Function(int) adder(int n) => (int x) => x + n;
final add5 = adder(5);
add5(3);   // 8

// Funciones anónimas
final square = (int x) => x * x;

// Funciones de orden superior built-in
[1, 2, 3].map((x) => x * 2);
[1, 2, 3].where((x) => x.isEven);
[1, 2, 3].reduce((a, b) => a + b);        // 6
[1, 2, 3].fold(0, (acc, x) => acc + x);   // 6
[1, 2, 3].any((x) => x > 2);
[1, 2, 3].every((x) => x > 0);
[[1, 2], [3, 4]].expand((x) => x);        // [1, 2, 3, 4]

// Recursión
int factorial(int n) => n <= 1 ? 1 : n * factorial(n - 1);

// Número variable de argumentos — usar List
int sum(List<int> numbers) => numbers.fold(0, (a, b) => a + b);
```

---

## 🛡️ Null Safety

```dart
// Tipos nullable vs non-nullable
String name = "Juan";       // Non-nullable
String? nickname = null;    // Nullable

// Operadores null-aware
String display = nickname ?? "Sin apodo";
int? len = nickname?.length;
nickname ??= "Default";
String upper = nickname!.toUpperCase();   // Aserción — lanza si null (evitar)

// Null-aware cascade
list?..add(1)..add(2);

// Null-aware index
Map<String, int>? map;
int? value = map?["key"];

// Acceso encadenado null-safe
String? city = user?.address?.city?.toUpperCase();

// late — inicialización diferida
class Service {
  late final Database _db;

  void init() => _db = Database();

  void query() => _db.run("SELECT *...");
  // Lanza LateInitializationError si se llama sin init()
}

// required — parámetro nombrado obligatorio
class Config {
  Config({required this.apiUrl, required this.timeout, this.retries = 3});
  final String apiUrl;
  final Duration timeout;
  final int retries;
}

// Null checks idiomáticos
// Forma 1: if null check con smart cast
if (nickname != null) {
  print(nickname.length);   // nickname promovido a String
}

// Forma 2: early return
void process() {
  final user = getUser();
  if (user == null) return;
  print(user.length);   // non-nullable aquí
}

// Forma 3: encadenamiento
final result = list
    .firstWhereOrNull((e) => e.id == id)
    ?.name ?? "Sin nombre";

// Nullable con colecciones
List<String?> withNulls = ["a", null, "b"];
List<String> clean = withNulls.whereType<String>().toList();
List<String> clean2 = withNulls.nonNulls.toList();   // Dart 3.3+

// Pattern matching con null
switch (value) {
  case String s: print(s.toUpperCase());
  case int n when n > 0: print("Positivo: $n");
  case null: print("Es null");
}
```

---

## 📦 Records y destructuring

```dart
// Records posicionales — (Type1, Type2)
(String, int) getUser() => ("Juan", 28);

final user = getUser();
print(user.$1);    // "Juan"
print(user.$2);    // 28

// Destructuring posicional
final (name, age) = getUser();
print("$name tiene $age años");

// Records con campos nombrados
({String name, int age, String email}) getProfile() =>
    (name: "Juan", age: 28, email: "juan@email.com");

final profile = getProfile();
print(profile.name);

// Destructuring nombrado
final (:name, :age, :email) = getProfile();

// Records mixtos
(int, {String label, bool active}) getStatus() =>
    (200, label: "OK", active: true);
final (code, :label, :active) = getStatus();

// Retornar múltiples valores (mejor que Map)
(double lat, double lng) getCoords() => (-25.2867, -57.6470);
final (lat, lng) = getCoords();

// Records en listas
final points = [(1.0, 2.0), (3.0, 4.0)];
for (final (x, y) in points) {
  print("x=$x, y=$y");
}

// Igualdad — mismos valores = iguales
print(("a", 1) == ("a", 1));          // true
print((x: 1, y: 2) == (y: 2, x: 1)); // true (nombres, no orden)

// Swap sin variable temporal
var a = 1, b = 2;
(a, b) = (b, a);

// Destructuring de listas y mapas
final [first, second, ...rest] = [1, 2, 3, 4, 5];
final {"name": n, "age": a2} = {"name": "Juan", "age": 28};
final [head, ...tail] = [1, 2, 3, 4];

// Destructuring anidado
final ((x1, y1), (x2, y2)) = ((1.0, 2.0), (3.0, 4.0));
```

---

## 🎯 Pattern matching

```dart
// switch expression — exhaustivo, retorna valor
String classify(Object value) => switch (value) {
  int n when n < 0   => "negativo",
  int n when n == 0  => "cero",
  int n              => "positivo",
  double d           => "decimal: $d",
  String s           => "texto: $s",
  null               => "nulo",
  _                  => "otro: $value",
};

// Patterns en switch statement
void process(Object? data) {
  switch (data) {
    case null:
      print("nulo");

    case int n when n > 100:
      print("número grande: $n");

    case String s when s.isEmpty:
      print("string vacío");

    case [int a, int b]:                          // List pattern exacto
      print("lista de dos: $a, $b");

    case [int first, ...]:                        // List con rest
      print("primer elemento: $first");

    case {"status": int code, "body": String body}: // Map pattern
      print("response $code: $body");

    case (String name, int age):                  // Record pattern
      print("$name tiene $age años");

    case MyClass(id: final id, name: final n):    // Object pattern
      print("objeto con id=$id, name=$n");
  }
}

// if-case — extraer valores sin switch completo
void handleResponse(Map<String, dynamic> json) {
  if (json case {"status": 200, "data": final List data}) {
    processData(data);
  } else if (json case {"error": final String msg}) {
    showError(msg);
  }
}

// Guards
switch (point) {
  case (int x, int y) when x == y:
    print("en la diagonal");
  case (int x, int y) when x > 0 && y > 0:
    print("primer cuadrante");
  case (int x, int y):
    print("punto: $x, $y");
}

// OR patterns
switch (status) {
  case "pending" || "processing": print("en curso");
  case "done" || "completed": print("finalizado");
  case "error" || "failed" || "cancelled": print("falló");
}

// Const patterns con enum
switch (direction) {
  case Direction.north || Direction.south: print("vertical");
  case Direction.east || Direction.west: print("horizontal");
}
```

---

## 🔒 Sealed classes y class modifiers

```dart
// sealed — clase cerrada, solo subclases en el mismo archivo/librería
// Permite exhaustividad en switch
sealed class Shape {}
class Circle extends Shape {
  final double radius;
  const Circle(this.radius);
}
class Rectangle extends Shape {
  final double width, height;
  const Rectangle(this.width, this.height);
}
class Triangle extends Shape {
  final double base, height;
  const Triangle(this.base, this.height);
}

// El compilador exige todos los casos — falla si falta uno
double area(Shape shape) => switch (shape) {
  Circle(:final radius)                    => pi * radius * radius,
  Rectangle(:final width, :final height)  => width * height,
  Triangle(:final base, :final height)    => base * height / 2,
};

// abstract — no se puede instanciar directamente
abstract class Animal {
  String get name;
  void makeSound();                            // Abstracto
  void breathe() => print("$name respira");   // Con implementación default
}

// interface — solo para implementar, no extender (Dart 3.x)
interface class Serializable {
  String toJson() => throw UnimplementedError();
}

// final — no se puede extender ni implementar fuera de la librería
final class Config {
  final String apiUrl;
  const Config(this.apiUrl);
}

// base — solo se puede extender (no implementar)
// Garantiza que la implementación base se hereda siempre
base class Repository {
  Future<void> save(Object data) async { /* impl base */ }
}

// mixin class — usable como mixin O como clase base
mixin class Validator {
  bool isValid(String value) => value.isNotEmpty;
}
class FormField extends Object with Validator {}
class StandaloneValidator extends Validator {}
```

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

---

## 📋 Listas

```dart
// Crear
List<int> empty = [];
List<int> fixed = List.filled(5, 0);           // [0, 0, 0, 0, 0]
List<int> generated = List.generate(5, (i) => i * 2);  // [0,2,4,6,8]
List<int> unmod = List.unmodifiable([1, 2, 3]);
var literal = [1, 2, 3, 4, 5];

// Acceso
list[0]; list.last; list.first;
list.length; list.isEmpty; list.isNotEmpty;

// Agregar / quitar
list.add(6);
list.addAll([7, 8, 9]);
list.insert(0, 0);
list.insertAll(0, [-2, -1]);
list.remove(3);            // Eliminar primera ocurrencia del valor
list.removeAt(0);
list.removeLast();
list.removeWhere((e) => e < 0);
list.clear();

// Búsqueda
list.contains(3);
list.indexOf(3);           // -1 si no existe
list.lastIndexOf(3);
list.indexWhere((e) => e > 5);
list.firstWhere((e) => e > 5, orElse: () => -1);
list.singleWhere((e) => e == 3);

// Ordenar
list.sort();
list.sort((a, b) => b.compareTo(a));   // Descendente
final sorted = [...list]..sort();      // Copia ordenada

// Sub-listas
list.sublist(1, 4);
list.take(3).toList();
list.skip(2).toList();
list.takeWhile((e) => e < 5).toList();
list.skipWhile((e) => e < 5).toList();

// Transformaciones
list.map((e) => e * 2).toList();
list.where((e) => e.isEven).toList();
list.expand((e) => [e, e * 2]).toList();
list.reduce((a, b) => a + b);
list.fold(0, (acc, e) => acc + e);
list.reversed.toList();
list.toSet();    // Elimina duplicados

// Iterar con índice
list.asMap().forEach((index, value) => print("$index: $value"));
list.indexed.forEach((p) => print("${p.$1}: ${p.$2}"));  // Dart 3.x

// Chequeos
list.any((e) => e > 5);
list.every((e) => e > 0);

// Join
["a", "b", "c"].join(", ");   // "a, b, c"
```

---

## 🗺️ Mapas

```dart
// Crear
Map<String, int> empty = {};
Map<String, int> literal = {"a": 1, "b": 2};
Map<String, int> fromEntries = Map.fromEntries([
  MapEntry("a", 1), MapEntry("b", 2),
]);
Map<String, int> fromIterables = Map.fromIterables(
  ["a", "b"], [1, 2],
);
final unmod = Map.unmodifiable(literal);

// Acceso
map["key"];                      // null si no existe
map["key"]!;                     // Lanza si null
map.containsKey("key");
map.containsValue(42);

// Agregar / modificar
map["newKey"] = 100;
map.putIfAbsent("key", () => 0);
map.update("key", (v) => v + 1);
map.update("key", (v) => v + 1, ifAbsent: () => 1);
map.addAll({"x": 10, "y": 20});

// Eliminar
map.remove("key");
map.removeWhere((k, v) => v < 0);
map.clear();

// Iterar
map.forEach((key, value) => print("$key: $value"));
for (final MapEntry(:key, :value) in map.entries) {
  print("$key: $value");
}
map.keys;      // Iterable<String>
map.values;    // Iterable<int>
map.entries;   // Iterable<MapEntry<String, int>>

// Transformar
map.map((k, v) => MapEntry(k.toUpperCase(), v * 2));

// Cast
final typed = (rawDynamic as Map).cast<String, int>();

// Null-aware
int safe = map["key"] ?? 0;
```

---

## 🔢 Sets

```dart
// Crear
Set<int> empty = {};
Set<int> literal = {1, 2, 3};
Set<int> from = Set.from([1, 2, 2, 3]);   // {1, 2, 3}
Set<int> unmod = Set.unmodifiable({1, 2, 3});

// Operaciones básicas
set.add(4);
set.addAll([5, 6, 7]);
set.remove(1);
set.contains(3);

// Operaciones de conjuntos
final a = {1, 2, 3, 4};
final b = {3, 4, 5, 6};

a.union(b);          // {1, 2, 3, 4, 5, 6}
a.intersection(b);   // {3, 4}
a.difference(b);     // {1, 2}

// Conversión
set.toList();
list.toSet();   // Elimina duplicados
```

---

## 🔁 Operaciones funcionales

```dart
final numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// map — transformar
final doubled = numbers.map((n) => n * 2).toList();

// where — filtrar
final evens = numbers.where((n) => n.isEven).toList();

// reduce — acumular (lanza si vacío)
final sum = numbers.reduce((acc, n) => acc + n);        // 55

// fold — con valor inicial (no lanza)
final sum2 = numbers.fold<int>(0, (acc, n) => acc + n); // 55
final product = numbers.fold<int>(1, (acc, n) => acc * n);

// expand — flatMap
final nested = [[1, 2], [3, 4], [5, 6]];
final flat = nested.expand((list) => list).toList();     // [1,2,3,4,5,6]

// any / every
numbers.any((n) => n > 9);       // true
numbers.every((n) => n > 0);     // true

// take / skip
numbers.take(3).toList();                       // [1, 2, 3]
numbers.skip(7).toList();                       // [8, 9, 10]
numbers.takeWhile((n) => n < 5).toList();       // [1, 2, 3, 4]
numbers.skipWhile((n) => n < 5).toList();       // [5, 6, 7, 8, 9, 10]

// Encadenamiento — lazy (no evalúa hasta iterar)
final result = numbers
    .where((n) => n.isEven)
    .map((n) => n * n)
    .where((n) => n > 10)
    .take(3)
    .toList();    // [16, 36, 64]

// zip con indexed (no existe built-in zip)
final a = [1, 2, 3];
final b = ["a", "b", "c"];
final zipped = a.indexed.map((e) => (e.$2, b[e.$1])).toList();
// [(1,"a"), (2,"b"), (3,"c")]
```

---

## 🌿 Spread, collection if, collection for

```dart
// Spread operator ...
final combined = [...list1, ...list2];
final withExtra = [0, ...list1, ...list2, 7];

// Null-aware spread ...?
List<int>? nullable;
final safe = [...?nullable, 1, 2, 3];    // [1, 2, 3]

// Map spread — el último gana en colisiones
final merged = {...defaults, ...userPrefs};

// Set spread
final union = {...set1, ...set2};

// Collection if
final menu = [
  "Inicio",
  if (isLoggedIn) "Perfil",
  if (isLoggedIn) "Configuración",
  if (!isLoggedIn) "Iniciar sesión",
];

// Collection if con else y spread
final items = [
  "Gratis",
  if (isPro) ...[
    "Feature Premium 1",
    "Feature Premium 2",
  ] else
    "Actualizar a Pro",
];

// Collection for
final squares = [for (int i = 1; i <= 5; i++) i * i];   // [1,4,9,16,25]

final users = [
  for (final name in ["Ana", "Juan", "María"]) User(name: name),
];

// Combinación
final activeItems = [
  for (final item in allItems)
    if (item.isActive) item.name,
];

// En mapas
final colorMap = {
  for (final color in colors) color.name: color.hex,
  if (includeClear) "clear": "#00000000",
};
```

---

## ⏳ Future — async/await

```dart
// Función async básica
Future<String> fetchUser(String id) async {
  await Future.delayed(const Duration(seconds: 1));
  return "User $id";
}

// Error handling
Future<String> safeGet(String url) async {
  try {
    final response = await httpClient.get(url);
    if (response.statusCode != 200) {
      throw HttpException("Error ${response.statusCode}");
    }
    return response.body;
  } on HttpException catch (e) {
    throw AppException("HTTP: ${e.message}");
  } on SocketException {
    throw AppException("Sin conexión");
  }
}

// then / catchError / whenComplete (callback style)
fetchUser("123")
    .then((user) => print("Usuario: $user"))
    .catchError((e) => print("Error: $e"))
    .whenComplete(() => print("Finalizado siempre"));

// Future.value / Future.error
final immediate = Future.value(42);
final failed = Future.error(Exception("falló"));

// Future.delayed
await Future.delayed(const Duration(milliseconds: 500));
final result = await Future.delayed(const Duration(seconds: 1), () => "tardío");

// Future.wait — paralelo, falla si uno falla
final [users, products] = await Future.wait([
  userRepo.getAll(),
  productRepo.getAll(),
]);

// Future.wait con eagerError: false — continuar aunque fallen
final results = await Future.wait(
  ids.map(fetchUser),
  eagerError: false,
);

// Future.any — el primero que resuelva
final fastest = await Future.any([server1.ping(), server2.ping()]);

// Future.timeout
final data = await fetchData().timeout(
  const Duration(seconds: 10),
  onTimeout: () => throw TimeoutException("Tiempo agotado"),
);

// Completer — crear Futures programáticamente
final completer = Completer<String>();
Timer(const Duration(seconds: 1), () {
  condition
      ? completer.complete("éxito")
      : completer.completeError(Exception("falló"));
});
final result2 = await completer.future;

// unawaited — fire-and-forget sin warning
unawaited(logAnalytics("event"));

// Async generator — yield múltiples valores
Stream<int> countDown(int from) async* {
  for (int i = from; i >= 0; i--) {
    await Future.delayed(const Duration(seconds: 1));
    yield i;
  }
}
```

---

## 🌊 Stream

```dart
// Stream básico con async*
Stream<int> numberStream() async* {
  for (int i = 0; i < 5; i++) {
    await Future.delayed(const Duration(milliseconds: 100));
    yield i;
  }
}

// Escuchar
final subscription = numberStream().listen(
  (value) => print("valor: $value"),
  onError: (error) => print("error: $error"),
  onDone: () => print("completado"),
  cancelOnError: false,
);

await subscription.cancel();
subscription.pause();
subscription.resume();

// await for — consume el stream
await for (final value in numberStream()) {
  print(value);
}

// StreamController
final controller = StreamController<String>();
controller.sink.add("mensaje 1");
controller.sink.addError(Exception("error"));
controller.close();
controller.stream.listen(print);

// Broadcast — múltiples listeners
final broadcast = StreamController<int>.broadcast();
broadcast.stream.listen((v) => print("L1: $v"));
broadcast.stream.listen((v) => print("L2: $v"));
broadcast.sink.add(42);

// Factories
Stream.fromIterable([1, 2, 3, 4, 5]);
Stream.fromFuture(Future.value(42));
Stream.periodic(const Duration(seconds: 1), (i) => i);
Stream.value(42);       // Single-value stream

// Transformaciones (lazy)
stream
    .map((e) => e * 2)
    .where((e) => e > 4)
    .take(3)
    .skip(1)
    .distinct();

// Stream a Future
await stream.first;
await stream.last;
await stream.single;
await stream.toList();
await stream.isEmpty;
await stream.length;

// StreamTransformer personalizado
final transformer = StreamTransformer<int, String>.fromHandlers(
  handleData: (data, sink) {
    if (data > 0) sink.add("positivo: $data");
  },
  handleError: (error, stack, sink) => sink.add("error manejado"),
);
numberStream().transform(transformer).listen(print);
```

---

## 🧵 Isolates

```dart
import 'dart:isolate';

// Isolate.run — la forma más simple (Dart 2.19+)
Future<List<int>> sortLargeList(List<int> data) async {
  return Isolate.run(() {
    final copy = List<int>.from(data);
    copy.sort();
    return copy;
  });
}

// Isolate.spawn — comunicación bidireccional
void _workerEntry(SendPort sendPort) {
  final receivePort = ReceivePort();
  sendPort.send(receivePort.sendPort);

  receivePort.listen((message) {
    if (message is int) sendPort.send(message * message);
    else if (message == 'close') receivePort.close();
  });
}

Future<void> useIsolate() async {
  final receivePort = ReceivePort();
  final isolate = await Isolate.spawn(_workerEntry, receivePort.sendPort);
  final sendPort = await receivePort.first as SendPort;

  sendPort.send(5);      // Enviar trabajo
  sendPort.send('close');

  isolate.kill(priority: Isolate.immediate);
  receivePort.close();
}

// Isolate.exit — enviar resultado y terminar
void _processData(SendPort sendPort) {
  final result = heavyComputation();
  Isolate.exit(sendPort, result);
}

// NOTA: solo se pueden pasar entre Isolates:
// primitivos, List, Map, Set, TransferableTypedData, SendPort
// Los objetos se COPIAN (sin memoria compartida)
```

---

## ⚠️ Excepciones

```dart
// throw — lanzar excepción
throw FormatException("JSON inválido: $rawJson");
throw ArgumentError.value(age, "age", "Debe ser positivo");
throw StateError("El objeto ya fue dispuesto");
throw Exception("Error genérico");

// try / catch / on / finally
Future<void> processFile(String path) async {
  File? file;
  try {
    file = File(path);
    final content = await file.readAsString();
    processContent(content);

  } on FileSystemException catch (e, stack) {
    print("Error de archivo: ${e.message}");
    log.error("FileSystemException", error: e, stackTrace: stack);

  } on FormatException catch (e) {
    print("Formato inválido: ${e.message}");
    rethrow;    // Relanzar la excepción original

  } on Exception catch (e) {
    print("Error: $e");

  } catch (e, stack) {
    print("Error inesperado: $e\n$stack");

  } finally {
    file?.deleteSync();    // Siempre se ejecuta
  }
}

// Excepciones personalizadas
class AppException implements Exception {
  final String message;
  final String? code;
  final Object? cause;

  const AppException(this.message, {this.code, this.cause});

  @override
  String toString() => "AppException($code): $message";
}

class NetworkException extends AppException {
  final int? statusCode;
  const NetworkException(super.message, {this.statusCode, super.cause})
      : super(code: "NETWORK_ERROR");
}

class ValidationException extends AppException {
  final Map<String, String> fieldErrors;
  const ValidationException(super.message, {required this.fieldErrors})
      : super(code: "VALIDATION_ERROR");
}

// assert — solo en modo debug
void setAge(int age) {
  assert(age >= 0, "La edad no puede ser negativa");
  assert(age < 150, "Edad irreal: $age");
  this.age = age;
}

// Zona de errores — atrapar errores async no manejados
runZonedGuarded(
  () async { runApp(const MyApp()); },
  (error, stack) {
    Crashlytics.instance.recordError(error, stack, fatal: true);
  },
);
```

---

## 🔍 Sistema de tipos avanzado

```dart
// Type checks y promotion
Object value = "hola";

value is String;         // true
value is int;            // false
value is! int;           // true

if (value is String) {
  print(value.length);   // value promovido a String aquí
}

// as — cast explícito
final str = value as String;   // Lanza CastError si falla

// safeCast
T? safeCast<T>(Object? value) => value is T ? value : null;

// dynamic — sin verificación de tipos en compilación
dynamic dyn = 42;
dyn = "string";          // Válido en compilación, puede romper en runtime

// Object? — alternativa a dynamic con seguridad
void printAnything(Object? value) {
  switch (value) {
    case null: print("null");
    case String s: print("String: $s");
    case int n: print("int: $n");
    default: print("otro: $value");
  }
}

// Callable classes — instancias usables como funciones
class Validator {
  final String pattern;
  const Validator(this.pattern);
  bool call(String value) => RegExp(pattern).hasMatch(value);
}

final isEmail = Validator(r'^[\w.-]+@[\w.-]+\.\w+$');
isEmail("test@email.com");   // Llama a call() — true

// Tearoff — referencia a método sin invocar
final fn = "hello".toUpperCase;
fn();                          // "HELLO"
final addMethod = 42.toString;
addMethod();                   // "42"
```

---

## 🎪 Funciones de primera clase y closures

```dart
// Funciones como valores
typedef BinaryOp = int Function(int, int);

BinaryOp getOperation(String op) => switch (op) {
  "+" => (a, b) => a + b,
  "-" => (a, b) => a - b,
  "*" => (a, b) => a * b,
  "/" => (a, b) => a ~/ b,
  _ => throw ArgumentError("Operación desconocida: $op"),
};

final add = getOperation("+");
add(3, 4);   // 7

// Closures — capturan variables del scope exterior
Function counter() {
  int count = 0;
  return () {
    count++;
    return count;
  };
}

final c1 = counter();
c1();   // 1
c1();   // 2
c1();   // 3

final c2 = counter();   // Contador independiente
c2();   // 1

// Currying
int Function(int) adder(int n) => (int x) => x + n;
final add5 = adder(5);
add5(3);   // 8
add5(10);  // 15

// Composición de funciones
T Function(A) compose<A, B, T>(T Function(B) f, B Function(A) g) =>
    (A x) => f(g(x));

final doubleAndStringify = compose<int, int, String>(
  (n) => n.toString(),
  (n) => n * 2,
);
doubleAndStringify(5);   // "10"

// Memoización
Map<A, R> Function(A) memoize<A, R>(R Function(A) fn) {
  final cache = <A, R>{};
  return (A arg) => cache.putIfAbsent(arg, () => fn(arg));
}

final cachedFactorial = memoize(factorial);
```

---

## 🔤 String — métodos esenciales

```dart
final s = "  Hola Mundo  ";

// Info
s.length;           // 14
s.isEmpty;
s.isNotEmpty;

// Transformación
s.trim();           // "Hola Mundo"
s.trimLeft();
s.trimRight();
s.toLowerCase();
s.toUpperCase();

// Búsqueda
s.contains("Mundo");
s.contains(RegExp(r'\d'));
s.startsWith("  Hola");
s.endsWith("  ");
s.indexOf("o");         // 4
s.lastIndexOf("o");     // 9

// Reemplazar
s.replaceAll("o", "0");
s.replaceFirst("o", "0");
s.replaceRange(2, 6, "X");
s.replaceAllMapped(
  RegExp(r'[aeiou]'),
  (m) => m.group(0)!.toUpperCase(),
);

// Dividir
"hola mundo".split(" ");         // ["hola", "mundo"]
s.split(RegExp(r'\s+'));
"a,b,,c".split(",");             // ["a", "b", "", "c"]

// Substring
s.substring(2, 6);              // "Hola"
s[0];                            // " "

// Padding
"42".padLeft(5, "0");           // "00042"
"hi".padRight(10, "-");         // "hi--------"

// Verificar
int.tryParse(s) != null;        // Es entero?
double.tryParse(s) != null;     // Es número?

// StringBuffer — eficiente para concatenar muchas strings
final buffer = StringBuffer();
buffer.write("Hola");
buffer.write(", ");
buffer.writeln("Mundo!");
buffer.writeAll(["a", "b", "c"], ", ");
final result = buffer.toString();

// Comparación
"abc".compareTo("abd");   // -1 (menor), 0 (igual), 1 (mayor)
```

---

## 📅 DateTime y Duration

```dart
// Crear
final now = DateTime.now();
final utc = DateTime.now().toUtc();
final specific = DateTime(2026, 1, 15, 10, 30, 0);
final fromMillis = DateTime.fromMillisecondsSinceEpoch(1000000);
final parsed = DateTime.parse("2026-01-15T10:30:00Z");
final tryParsed = DateTime.tryParse("invalid");   // null

// Propiedades
now.year; now.month; now.day;
now.hour; now.minute; now.second;
now.millisecond; now.microsecond;
now.weekday;   // 1=Lun (DateTime.monday) .. 7=Dom
now.isUtc;
now.millisecondsSinceEpoch;

// Comparar
now.isBefore(specific);
now.isAfter(specific);
now.isAtSameMomentAs(specific);

// Manipular
now.add(const Duration(days: 7));
now.subtract(const Duration(hours: 24));
now.copyWith(hour: 0, minute: 0, second: 0);

// Diferencia → Duration
final diff = end.difference(start);
diff.inDays; diff.inHours; diff.inMinutes;
diff.inSeconds; diff.inMilliseconds; diff.abs();

// Duration
const dur = Duration(days: 1, hours: 2, minutes: 30);
dur.inHours;          // 26
dur.inMinutes;        // 1590
dur + Duration(minutes: 30);
dur * 2;
dur.isNegative;

// Formateo básico (sin package:intl)
"${now.day.toString().padLeft(2,'0')}/${now.month.toString().padLeft(2,'0')}/${now.year}";
now.toIso8601String();   // "2026-01-15T10:30:00.000"
```

---

## 📐 Matemáticas — dart:math

```dart
import 'dart:math';

// Constantes
pi;       // 3.141592653589793
e;        // 2.718281828459045
sqrt2;    // 1.4142135623730951

// Funciones
sqrt(16);          // 4.0
pow(2, 10);        // 1024
exp(1);            // e
log(e);            // 1.0 — logaritmo natural
sin(pi / 2);       // 1.0
cos(0);            // 1.0
tan(pi / 4);       // 1.0
asin(1);           // pi/2
acos(1);           // 0.0
atan(1);           // pi/4
atan2(1, 1);       // pi/4

min(3, 5);         // 3
max(3, 5);         // 5

// Clamp — limitar valor a rango
42.clamp(0, 100);    // 42
150.clamp(0, 100);   // 100
(-5).clamp(0, 100);  // 0

// Redondeo
3.7.round();       // 4
3.7.floor();       // 3
3.7.ceil();        // 4
3.7.truncate();    // 3

// Números aleatorios
final random = Random();
random.nextInt(100);     // 0..99
random.nextDouble();     // 0.0..1.0
random.nextBool();

// Aleatorio seguro (criptografía)
final secureRandom = Random.secure();
secureRandom.nextInt(256);

// Lista aleatoria
List.generate(10, (_) => random.nextInt(100));
```

---

## 🔎 Expresiones regulares — RegExp

```dart
// Crear
final emailRegex = RegExp(r'^[\w\.-]+@[\w\.-]+\.\w{2,}$');
final phoneRegex = RegExp(r'^\+?[1-9]\d{7,14}$');
final urlRegex = RegExp(r'https?://[^\s]+');
final numberRegex = RegExp(r'\d+');

// hasMatch
emailRegex.hasMatch("test@email.com");   // true
emailRegex.hasMatch("no-email");         // false

// firstMatch
final match = numberRegex.firstMatch("abc 123 def");
match?.group(0);    // "123"
match?.start;       // 4
match?.end;         // 7

// allMatches
final numbers = numberRegex
    .allMatches("abc 123 def 456")
    .map((m) => int.parse(m.group(0)!))
    .toList();   // [123, 456]

// Grupos de captura
final dateRegex = RegExp(r'(\d{4})-(\d{2})-(\d{2})');
final dm = dateRegex.firstMatch("2026-01-15");
dm?.group(1);   // "2026"
dm?.group(2);   // "01"
dm?.group(3);   // "15"

// Grupos nombrados
final namedDate = RegExp(r'(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})');
final nm = namedDate.firstMatch("2026-01-15");
nm?.namedGroup("year");    // "2026"

// replaceAllMapped
"hello world".replaceAllMapped(
  RegExp(r'\b\w'),
  (m) => m.group(0)!.toUpperCase(),
);   // "Hello World"

// Flags
RegExp(r'hello', caseSensitive: false);   // Case insensitive
RegExp(r'^inicio', multiLine: true);      // ^ y $ por línea
RegExp(r'texto.punto', dotAll: true);     // . incluye \n

// Validadores comunes
class Validators {
  static final _email = RegExp(r'^[\w\.-]+@[\w\.-]+\.\w{2,}$');
  static final _url = RegExp(r'^https?://[^\s/$.?#].[^\s]*$');
  static final _alphanumeric = RegExp(r'^[a-zA-Z0-9]+$');
  static final _uuid = RegExp(
    r'^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$',
    caseSensitive: false,
  );

  static bool isEmail(String s) => _email.hasMatch(s);
  static bool isUrl(String s) => _url.hasMatch(s);
  static bool isAlphanumeric(String s) => _alphanumeric.hasMatch(s);
  static bool isUuid(String s) => _uuid.hasMatch(s);
}
```

---

## 💾 dart:io — archivos y directorios

```dart
import 'dart:io';

// Archivos
final file = File('datos.txt');

await file.exists();
file.existsSync();

// Leer
final content = await file.readAsString();
final lines = await file.readAsLines();
final bytes = await file.readAsBytes();

// Stream para archivos grandes
await for (final line in file.openRead()
    .transform(utf8.decoder)
    .transform(const LineSplitter())) {
  print(line);
}

// Escribir
await file.writeAsString("contenido");
await file.writeAsString("más\n", mode: FileMode.append);
await file.writeAsBytes(bytes);

// Operaciones
await file.copy('/destino/datos.txt');
await file.rename('/nuevo/path.txt');
await file.delete();
await file.create(recursive: true);

// Metadata
final stat = await file.stat();
stat.size; stat.modified; stat.type;

// Directorios
final dir = Directory('/mi/directorio');
await dir.create(recursive: true);
await dir.delete(recursive: true);

await for (final entity in dir.list(recursive: false)) {
  if (entity is File) print("Archivo: ${entity.path}");
  if (entity is Directory) print("Dir: ${entity.path}");
}

Directory.systemTemp;
Directory.current;

// Platform — info del sistema
Platform.operatingSystem;   // "linux", "macos", "windows", "android", "ios"
Platform.isLinux; Platform.isMacOS; Platform.isWindows;
Platform.isAndroid; Platform.isIOS;
Platform.version;              // Versión de Dart
Platform.numberOfProcessors;
Platform.localeName;           // "es_PY"
Platform.environment['HOME'];  // Variables de entorno

// stdin / stdout
stdout.writeln("Hola desde Dart");
stderr.writeln("Error");
final input = stdin.readLineSync();

// Procesos
final result = await Process.run('ls', ['-la']);
print(result.stdout);
print(result.exitCode);

final process = await Process.start('python3', ['script.py']);
process.stdout.transform(utf8.decoder).listen(print);
await process.exitCode;
```

---

## 🔄 dart:convert — JSON y codificación

```dart
import 'dart:convert';

// JSON encode/decode
final map = {'name': 'Juan', 'age': 28, 'active': true};
final jsonString = jsonEncode(map);
final decoded = jsonDecode(jsonString) as Map<String, dynamic>;

// Pretty print
const encoder = JsonEncoder.withIndent('  ');
final pretty = encoder.convert(map);

// Decode tipado
final name = decoded['name'] as String;
final age = decoded['age'] as int;

// Listas JSON
final list = (jsonDecode('[1,2,3]') as List).cast<int>();

// JSON con clases
class User {
  final String name;
  final int age;
  User({required this.name, required this.age});

  factory User.fromJson(Map<String, dynamic> j) =>
      User(name: j['name'] as String, age: j['age'] as int);

  Map<String, dynamic> toJson() => {'name': name, 'age': age};
}

// Encoder custom (para tipos no serializables)
final customEncoder = JsonEncoder.withIndent('  ', (obj) {
  if (obj is DateTime) return obj.toIso8601String();
  if (obj is Uri) return obj.toString();
  throw UnsupportedError("No soportado: ${obj.runtimeType}");
});

// UTF-8
final bytes = utf8.encode("Hola 🌍");
final str = utf8.decode(bytes);

// Base64
final encoded = base64Encode(utf8.encode("texto secreto"));
final decoded2 = utf8.decode(base64Decode(encoded));

// Base64 URL-safe
final urlEncoded = base64Url.encode(utf8.encode("texto"));
final urlDecoded = utf8.decode(base64Url.decode(urlEncoded));

// HTML escape
final safe = HtmlEscape().convert("<script>alert('xss')</script>");

// LineSplitter
final lineList = const LineSplitter().convert("a\nb\nc");

// Codecs encadenados
final gzippedUtf8 = utf8.fuse(gzip);
final compressed = gzippedUtf8.encode("texto largo a comprimir");
final original = gzippedUtf8.decode(compressed);
```

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
