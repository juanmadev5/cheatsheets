# Flutter — Almacenamiento y compartir archivos

---

## 📁 path_provider — directorios de la app

```dart
// pubspec.yaml
// path_provider: ^2.1.6

import 'package:path_provider/path_provider.dart';

// Documentos — persiste entre sesiones, el usuario NO lo ve directamente
final docsDir = await getApplicationDocumentsDirectory();

// Soporte de la app — datos internos que no debe tocar el usuario
final supportDir = await getApplicationSupportDirectory();

// Caché — el SO puede borrarlo cuando necesita espacio
final cacheDir = await getApplicationCacheDirectory();
final tempDir = await getTemporaryDirectory();

// Downloads del usuario (solo desktop y Android)
final downloadsDir = await getDownloadsDirectory();

// Escribir y leer un archivo
Future<File> saveTextFile(String filename, String content) async {
  final dir = await getApplicationDocumentsDirectory();
  final file = File('${dir.path}/$filename');
  return file.writeAsString(content);
}

Future<String?> readTextFile(String filename) async {
  final dir = await getApplicationDocumentsDirectory();
  final file = File('${dir.path}/$filename');
  return file.existsSync() ? file.readAsString() : null;
}
```

---

## 🖼️ gal — guardar imágenes y videos en la galería

```dart
// pubspec.yaml
// gal: ^2.3.2

import 'package:gal/gal.dart';

// Verificar/pedir acceso antes de guardar (Android 13+ / iOS)
Future<bool> ensureGalleryAccess() async {
  final hasAccess = await Gal.hasAccess();
  if (hasAccess) return true;
  return Gal.requestAccess();
}

// Guardar desde un path de archivo
Future<void> saveImageToGallery(String filePath) async {
  if (!await ensureGalleryAccess()) return;
  await Gal.putImage(filePath, album: 'MiApp');
}

// Guardar desde bytes (ej: imagen procesada en memoria)
Future<void> saveBytesToGallery(Uint8List bytes) async {
  if (!await ensureGalleryAccess()) return;
  await Gal.putImageBytes(bytes, album: 'MiApp');
}

// Guardar un video
Future<void> saveVideoToGallery(String filePath) async {
  if (!await ensureGalleryAccess()) return;
  await Gal.putVideo(filePath, album: 'MiApp');
}
```

```xml
<!-- Android: AndroidManifest.xml — no requiere permisos extra en Android 10+ (usa MediaStore) -->

<!-- iOS: ios/Runner/Info.plist -->
<key>NSPhotoLibraryAddUsageDescription</key>
<string>Necesitamos guardar tus fotos en la galería</string>
```

---

## 📂 file_picker — elegir archivos del dispositivo

```dart
// pubspec.yaml
// file_picker: ^11.0.2

import 'package:file_picker/file_picker.dart';

// Elegir un solo archivo
Future<File?> pickFile() async {
  final result = await FilePicker.platform.pickFiles();
  if (result == null) return null; // Usuario canceló
  return File(result.files.single.path!);
}

// Elegir múltiples archivos con filtro de extensión
Future<List<File>> pickDocuments() async {
  final result = await FilePicker.platform.pickFiles(
    allowMultiple: true,
    type: FileType.custom,
    allowedExtensions: ['pdf', 'doc', 'docx'],
  );
  if (result == null) return [];
  return result.paths.whereType<String>().map(File.new).toList();
}

// Elegir imágenes/videos (usa el picker nativo de medios)
FilePicker.platform.pickFiles(type: FileType.image);
FilePicker.platform.pickFiles(type: FileType.video);

// Web — usar los bytes en vez del path (no hay filesystem)
final result = await FilePicker.platform.pickFiles(withData: true);
final bytes = result?.files.single.bytes;

// Elegir un directorio completo
final dirPath = await FilePicker.platform.getDirectoryPath();
```

---

## 📤 share_plus — compartir contenido

```dart
// pubspec.yaml
// share_plus: ^13.2.1

import 'package:share_plus/share_plus.dart';
import 'package:cross_file/cross_file.dart';

// Compartir texto/link — abre el selector nativo de apps (ACTION_SEND / UIActivityViewController)
await SharePlus.instance.share(
  ShareParams(text: 'Mirá esta app: https://miapp.com'),
);

// Compartir un archivo (ej: una imagen generada)
final result = await SharePlus.instance.share(
  ShareParams(
    text: 'Mi resultado',
    files: [XFile('${directory.path}/resultado.png')],
  ),
);

if (result.status == ShareResultStatus.success) {
  // El usuario completó el share
}

// En iPad, share_plus requiere un sharePositionOrigin para el popover
await SharePlus.instance.share(
  ShareParams(
    text: 'Contenido',
    sharePositionOrigin: buttonKey.globalPaintBounds,
  ),
);
```

---

## 🔗 url_launcher — abrir URLs y otras apps

```dart
// pubspec.yaml
// url_launcher: ^6.3.2

import 'package:url_launcher/url_launcher.dart';

// Abrir una URL en el navegador
Future<void> openWebsite(String url) async {
  final uri = Uri.parse(url);
  if (!await launchUrl(uri)) {
    throw Exception('No se pudo abrir $url');
  }
}

// Abrir dentro de la app (in-app browser) en vez de salir a Chrome/Safari
launchUrl(uri, mode: LaunchMode.inAppWebView);

// Marcar un teléfono
launchUrl(Uri(scheme: 'tel', path: '+595981123456'));

// Enviar un email
launchUrl(Uri(
  scheme: 'mailto',
  path: 'contacto@miapp.com',
  query: 'subject=Consulta&body=Hola',
));

// Abrir WhatsApp con mensaje predefinido
launchUrl(Uri.parse('https://wa.me/595981123456?text=Hola'));

// Verificar si hay una app capaz de manejar la URL antes de intentar abrirla
if (await canLaunchUrl(uri)) {
  await launchUrl(uri);
}
```

```xml
<!-- iOS: ios/Runner/Info.plist — declarar esquemas custom que se van a verificar con canLaunchUrl -->
<key>LSApplicationQueriesSchemes</key>
<array>
  <string>whatsapp</string>
  <string>tel</string>
  <string>mailto</string>
</array>
```
