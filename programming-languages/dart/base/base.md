# Dart — Base

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
int big = 1_000_000;         // Separador de dígitos, no solo miles (Dart 3.6+)
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

// Dart no tiene "if" como expresión — usar el operador ternario (?:)
final label = score >= 90 ? "Excelente" : "Mejorable";

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
