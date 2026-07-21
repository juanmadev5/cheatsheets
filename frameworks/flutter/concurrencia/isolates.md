# Flutter — Concurrencia (Isolates y compute)

---

## ⚡ Isolates y compute

```dart
// compute() — operación pesada sin bloquear UI thread
// Ideal para: parsear JSON grande, encriptar, redimensionar imágenes

// Función top-level o static (requisito de compute)
List<Product> _parseProducts(String jsonString) {
  final list = jsonDecode(jsonString) as List;
  return list.map((e) => Product.fromJson(e as JsonMap)).toList();
}

// Usar en provider/ViewModel
final products = await compute(_parseProducts, response.body);

// Isolate.run (Dart 2.19+) — más simple que Isolate.spawn
Future<Uint8List> processImage(Uint8List imageBytes) async {
  return Isolate.run(() {
    final image = img.decodeImage(imageBytes)!;
    final resized = img.copyResize(image, width: 800);
    return Uint8List.fromList(img.encodeJpg(resized, quality: 85));
  });
}

// Isolate con comunicación bidireccional (para tareas largas)
class HeavyTaskIsolate {
  late Isolate _isolate;
  late SendPort _sendPort;
  final _receivePort = ReceivePort();

  Future<void> start() async {
    _isolate = await Isolate.spawn(_isolateEntry, _receivePort.sendPort);
    _sendPort = await _receivePort.first as SendPort;
  }

  Future<dynamic> runTask(dynamic data) async {
    final responsePort = ReceivePort();
    _sendPort.send([responsePort.sendPort, data]);
    return responsePort.first;
  }

  void stop() {
    _isolate.kill();
    _receivePort.close();
  }

  static void _isolateEntry(SendPort sendPort) {
    final port = ReceivePort();
    sendPort.send(port.sendPort);

    port.listen((message) {
      final responsePort = message[0] as SendPort;
      final data = message[1];
      // Procesar data...
      responsePort.send(result);
    });
  }
}
```


