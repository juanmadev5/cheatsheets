# Dart — Dart 3 Moderno

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

## 🔹 Dot shorthands

```dart
// Dot shorthands (Dart 3.10+) — omite el nombre del tipo cuando el
// compilador puede inferirlo del contexto (requiere sdk: ^3.10.0)

// Enum
enum Status { running, stopped }
Status current = .running;             // en vez de Status.running

// Constructores y miembros estáticos
Point origin = .origin();              // en vez de Point.origin()
List<int> zeros = .filled(5, 0);       // en vez de List.filled(5, 0)
int port = .parse('8080');             // en vez de int.parse('8080')

// En switch expression
String describe(Status s) => switch (s) {
  .running => "corriendo",
  .stopped => "detenido",
};

// Útil en argumentos nombrados donde el tipo ya es conocido (ej. Flutter)
// Row(mainAxisAlignment: .center, children: [...])
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

## 🎁 Extension types

```dart
// Extension type (Dart 3.3+) — wrapper de costo cero, solo existe en
// tiempo de compilación. A diferencia de "extension", crea un tipo NUEVO
// y distinto (no agrega miembros al tipo original).
extension type SafeId(int value) {
  bool get isValid => value > 0;
  operator <(SafeId other) => value < other.value;
}

void main() {
  final id = SafeId(42);
  id.isValid;          // true
  id < SafeId(41);      // false
  // id + 10;           // ❌ Error de compilación — '+' no existe en SafeId
}

// El tipo se "borra" al compilar — en runtime sigue siendo el tipo
// representado (int). No hay objeto wrapper real, ni overhead de memoria.
SafeId id = SafeId(1);
print(id.runtimeType);   // int — no "SafeId"

// Diferencia clave con "extension" normal:
extension IntShout on int {
  String shout() => "$this!!!";      // Se agrega a TODOS los int
}
extension type ShoutInt(int value) {
  String shout() => "$value!!!";     // Solo existe para el tipo ShoutInt
}

void demo() {
  5.shout();              // ✅ funciona en cualquier int (extension normal)
  ShoutInt(5).shout();     // ✅ funciona solo en ShoutInt
  int x = 5;
  // x = ShoutInt(5);      // ❌ Error — tipos distintos, no son intercambiables
}

// implements — extension type "transparente", expone además los
// miembros del tipo representado (útil para no perder la API original)
extension type Meters(double value) implements double {
  Meters get toFeet => Meters(value * 3.28084);
}

void useTransparent() {
  final m = Meters(10);
  m + 5;          // ✅ hereda el operator+ de double
  m.toFeet;        // ✅ miembro propio
}

// Caso de uso típico: type-safety para IDs/unidades sin overhead
extension type UserId(int value) {}
extension type ProductId(int value) {}

void findUser(UserId id) { /* ... */ }
// findUser(ProductId(1));   // ❌ Error de compilación — no se pueden confundir
findUser(UserId(1));         // ✅

// NOTA: extension types no pueden tener campos propios (solo la
// representación) ni miembros abstractos — solo métodos, getters,
// setters y operadores calculados a partir del valor representado.
```
