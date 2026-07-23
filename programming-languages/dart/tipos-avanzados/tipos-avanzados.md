# Dart — Sistema de tipos y patrones avanzados

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
