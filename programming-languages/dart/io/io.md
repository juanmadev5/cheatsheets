# Dart — Entrada/Salida — dart:io y dart:convert

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
