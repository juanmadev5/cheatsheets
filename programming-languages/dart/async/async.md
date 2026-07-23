# Dart — Async — Future, Stream e Isolates

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
