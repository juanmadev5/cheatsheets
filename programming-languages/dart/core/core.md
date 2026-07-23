# Dart — Dart Core — String, DateTime, Math y RegExp

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
