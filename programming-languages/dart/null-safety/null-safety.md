# Dart — Null Safety

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
