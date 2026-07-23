# Dart — Manejo de errores

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
