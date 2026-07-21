# Android Jetpack Compose — Almacenamiento

---

## 📂 Almacenamiento — leer y escribir en directorios públicos

### Resumen de opciones de almacenamiento

| Opción | Dónde | Accesible por otras apps | Se borra al desinstalar |
|---|---|---|---|
| Internal files | `/data/data/<pkg>/files/` | No | Sí |
| Internal cache | `/data/data/<pkg>/cache/` | No | Sí (+ limpiado por el SO) |
| External privado | `sdcard/Android/data/<pkg>/` | No (Android 10+) | Sí |
| **MediaStore** | `sdcard/Pictures/`, `Music/`, etc. | **Sí** | No (persiste) |
| **SAF (Storage Access Framework)** | Cualquier directorio | **Con permiso del usuario** | No |
| Downloads público | `sdcard/Download/` | Sí | No |

---

### MediaStore — escribir en directorios públicos (imágenes, video, audio)

MediaStore es el sistema recomendado para guardar archivos en directorios públicos como `Pictures`, `Movies`, `Music`, `Downloads` sin necesitar permisos de almacenamiento en Android 10+.

```kotlin
// Guardar imagen en Pictures/NombreApp/
suspend fun saveImageToGallery(context: Context, bitmap: Bitmap, filename: String): Uri? {
    val contentValues = ContentValues().apply {
        put(MediaStore.Images.Media.DISPLAY_NAME, "$filename.jpg")
        put(MediaStore.Images.Media.MIME_TYPE, "image/jpeg")
        put(MediaStore.Images.Media.RELATIVE_PATH, "Pictures/MiApp") // Subcarpeta dentro de Pictures
        put(MediaStore.Images.Media.IS_PENDING, 1)  // Marcar como pendiente durante escritura
    }

    val resolver = context.contentResolver
    val uri = resolver.insert(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, contentValues)
        ?: return null

    return try {
        resolver.openOutputStream(uri)?.use { stream ->
            bitmap.compress(Bitmap.CompressFormat.JPEG, 90, stream)
        }
        // Marcar como completo — lo hace visible en la galería
        contentValues.clear()
        contentValues.put(MediaStore.Images.Media.IS_PENDING, 0)
        resolver.update(uri, contentValues, null, null)
        uri
    } catch (e: Exception) {
        resolver.delete(uri, null, null) // Limpiar si falla
        null
    }
}

// Guardar video en Movies/NombreApp/
suspend fun saveVideoToMovies(context: Context, sourceFile: File, filename: String): Uri? {
    val contentValues = ContentValues().apply {
        put(MediaStore.Video.Media.DISPLAY_NAME, "$filename.mp4")
        put(MediaStore.Video.Media.MIME_TYPE, "video/mp4")
        put(MediaStore.Video.Media.RELATIVE_PATH, "Movies/MiApp")
        put(MediaStore.Video.Media.IS_PENDING, 1)
    }

    val resolver = context.contentResolver
    val uri = resolver.insert(MediaStore.Video.Media.EXTERNAL_CONTENT_URI, contentValues)
        ?: return null

    return try {
        resolver.openOutputStream(uri)?.use { output ->
            sourceFile.inputStream().use { input -> input.copyTo(output) }
        }
        contentValues.clear()
        contentValues.put(MediaStore.Video.Media.IS_PENDING, 0)
        resolver.update(uri, contentValues, null, null)
        uri
    } catch (e: Exception) {
        resolver.delete(uri, null, null)
        null
    }
}

// Guardar archivo en Downloads/
suspend fun saveToDownloads(context: Context, content: String, filename: String): Uri? {
    val contentValues = ContentValues().apply {
        put(MediaStore.Downloads.DISPLAY_NAME, filename)           // Ej: "reporte.csv"
        put(MediaStore.Downloads.MIME_TYPE, "text/csv")
        put(MediaStore.Downloads.RELATIVE_PATH, "Download/MiApp") // Subcarpeta en Downloads
        put(MediaStore.Downloads.IS_PENDING, 1)
    }

    val resolver = context.contentResolver
    val uri = resolver.insert(MediaStore.Downloads.EXTERNAL_CONTENT_URI, contentValues)
        ?: return null

    return try {
        resolver.openOutputStream(uri)?.use { stream ->
            stream.write(content.toByteArray(Charsets.UTF_8))
        }
        contentValues.clear()
        contentValues.put(MediaStore.Downloads.IS_PENDING, 0)
        resolver.update(uri, contentValues, null, null)
        uri
    } catch (e: Exception) {
        resolver.delete(uri, null, null)
        null
    }
}
```

---

### MediaStore — leer archivos públicos

```kotlin
// Leer imágenes de la galería con permisos (Android 13+: READ_MEDIA_IMAGES)
fun queryImages(context: Context): List<MediaImage> {
    val images = mutableListOf<MediaImage>()
    val projection = arrayOf(
        MediaStore.Images.Media._ID,
        MediaStore.Images.Media.DISPLAY_NAME,
        MediaStore.Images.Media.DATE_ADDED,
        MediaStore.Images.Media.SIZE,
        MediaStore.Images.Media.MIME_TYPE,
    )
    val sortOrder = "${MediaStore.Images.Media.DATE_ADDED} DESC"

    context.contentResolver.query(
        MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
        projection,
        null,   // selection (WHERE)
        null,   // selectionArgs
        sortOrder,
    )?.use { cursor ->
        val idColumn = cursor.getColumnIndexOrThrow(MediaStore.Images.Media._ID)
        val nameColumn = cursor.getColumnIndexOrThrow(MediaStore.Images.Media.DISPLAY_NAME)
        val dateColumn = cursor.getColumnIndexOrThrow(MediaStore.Images.Media.DATE_ADDED)

        while (cursor.moveToNext()) {
            val id = cursor.getLong(idColumn)
            val name = cursor.getString(nameColumn)
            val date = cursor.getLong(dateColumn)
            val uri = ContentUris.withAppendedId(
                MediaStore.Images.Media.EXTERNAL_CONTENT_URI, id
            )
            images.add(MediaImage(uri = uri, name = name, dateAdded = date))
        }
    }
    return images
}

// Filtrar por directorio o tipo
context.contentResolver.query(
    MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
    projection,
    selection = "${MediaStore.Images.Media.RELATIVE_PATH} LIKE ?",
    selectionArgs = arrayOf("Pictures/MiApp/%"),
    sortOrder = null,
)

// Leer contenido de un archivo por URI
fun readTextFromUri(context: Context, uri: Uri): String? {
    return context.contentResolver.openInputStream(uri)?.use { stream ->
        stream.bufferedReader().readText()
    }
}

// Obtener bitmap de una URI (Coil lo hace automáticamente, pero para uso manual)
fun loadBitmapFromUri(context: Context, uri: Uri): Bitmap? {
    return if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
        val source = ImageDecoder.createSource(context.contentResolver, uri)
        ImageDecoder.decodeBitmap(source)
    } else {
        @Suppress("DEPRECATION")
        MediaStore.Images.Media.getBitmap(context.contentResolver, uri)
    }
}
```

---

### SAF (Storage Access Framework) — acceso a cualquier directorio

El SAF permite al usuario elegir archivos o carpetas desde cualquier proveedor (almacenamiento local, Google Drive, etc.) sin que la app necesite permisos de almacenamiento.

```kotlin
// Abrir un archivo para LEER (el usuario elige)
val openFileLauncher = rememberLauncherForActivityResult(
    ActivityResultContracts.OpenDocument()
) { uri: Uri? ->
    uri ?: return@rememberLauncherForActivityResult
    // URI persistente — pedir permiso de persistencia
    context.contentResolver.takePersistableUriPermission(
        uri,
        Intent.FLAG_GRANT_READ_URI_PERMISSION,
    )
    val content = context.contentResolver.openInputStream(uri)?.use {
        it.bufferedReader().readText()
    }
}
// Abrir solo PDFs y documentos
openFileLauncher.launch(arrayOf("application/pdf", "text/plain", "text/csv"))

// Abrir múltiples archivos
val openMultipleLauncher = rememberLauncherForActivityResult(
    ActivityResultContracts.OpenMultipleDocuments()
) { uris: List<Uri> ->
    uris.forEach { uri -> /* procesar cada archivo */ }
}
openMultipleLauncher.launch(arrayOf("image/*"))

// Crear un archivo nuevo (el usuario elige dónde guardar)
val createFileLauncher = rememberLauncherForActivityResult(
    ActivityResultContracts.CreateDocument("text/plain")
) { uri: Uri? ->
    uri ?: return@rememberLauncherForActivityResult
    context.contentResolver.openOutputStream(uri)?.use { stream ->
        stream.write("Contenido del archivo".toByteArray())
    }
}
createFileLauncher.launch("mi_archivo.txt")  // Nombre sugerido

// Elegir directorio completo (acceso persistente a una carpeta)
val openDirLauncher = rememberLauncherForActivityResult(
    ActivityResultContracts.OpenDocumentTree()
) { treeUri: Uri? ->
    treeUri ?: return@rememberLauncherForActivityResult
    // Pedir permisos persistentes (sobrevive reinicios)
    context.contentResolver.takePersistableUriPermission(
        treeUri,
        Intent.FLAG_GRANT_READ_URI_PERMISSION or Intent.FLAG_GRANT_WRITE_URI_PERMISSION,
    )
    // Listar archivos del directorio
    val docTree = DocumentFile.fromTreeUri(context, treeUri)
    docTree?.listFiles()?.forEach { file ->
        Log.d("SAF", "Archivo: ${file.name}, tipo: ${file.type}")
    }
}
openDirLauncher.launch(null) // null = directorio raíz por defecto

// Leer/escribir en un directorio con permisos persistentes (sesiones futuras)
fun writeToPersistedDir(context: Context, treeUri: Uri, filename: String, content: String) {
    val docTree = DocumentFile.fromTreeUri(context, treeUri) ?: return
    val newFile = docTree.createFile("text/plain", filename) ?: return
    context.contentResolver.openOutputStream(newFile.uri)?.use { stream ->
        stream.write(content.toByteArray())
    }
}

// Verificar permisos persistentes ya otorgados
fun getPersistedDirectories(context: Context): List<Uri> {
    return context.contentResolver.persistedUriPermissions
        .filter { it.isReadPermission && it.isWritePermission }
        .map { it.uri }
}
```

---

### FileProvider — compartir archivos internos con otras apps

FileProvider permite compartir archivos del almacenamiento privado de tu app (internal/external) con otras apps de forma segura, sin exponer rutas del filesystem.

```xml
<!-- AndroidManifest.xml -->
<provider
    android:name="androidx.core.content.FileProvider"
    android:authorities="${applicationId}.fileprovider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/file_paths" />
</provider>
```

```xml
<!-- res/xml/file_paths.xml -->
<?xml version="1.0" encoding="utf-8"?>
<paths>
    <!-- Almacenamiento interno: context.filesDir -->
    <files-path name="internal_files" path="." />

    <!-- Cache interno: context.cacheDir -->
    <cache-path name="internal_cache" path="." />

    <!-- Almacenamiento externo privado: context.getExternalFilesDir(null) -->
    <external-files-path name="external_files" path="." />

    <!-- Cache externo: context.externalCacheDir -->
    <external-cache-path name="external_cache" path="." />

    <!-- Directorio específico dentro de files -->
    <files-path name="shared_images" path="images/" />
</paths>
```

```kotlin
// Obtener URI segura para compartir un archivo interno
fun getShareableUri(context: Context, file: File): Uri {
    return FileProvider.getUriForFile(
        context,
        "${context.packageName}.fileprovider",
        file,
    )
}

// Compartir un PDF generado internamente
val pdfFile = File(context.filesDir, "reporte.pdf")
// ... generar el PDF ...
val uri = getShareableUri(context, pdfFile)

val shareIntent = Intent(Intent.ACTION_SEND).apply {
    type = "application/pdf"
    putExtra(Intent.EXTRA_STREAM, uri)
    addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION)  // OBLIGATORIO para FileProvider
}
startActivity(Intent.createChooser(shareIntent, "Compartir PDF"))

// Abrir archivo con otra app (visor de PDF, editor, etc.)
val viewIntent = Intent(Intent.ACTION_VIEW).apply {
    setDataAndType(uri, "application/pdf")
    addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION)
}
startActivity(viewIntent)

// Capturar foto con la cámara y guardar en archivo interno
val photoFile = File(context.cacheDir, "temp_photo.jpg")
val photoUri = getShareableUri(context, photoFile)

val cameraLauncher = rememberLauncherForActivityResult(
    ActivityResultContracts.TakePicture()
) { success ->
    if (success) {
        // La foto fue guardada en photoFile
        val bitmap = BitmapFactory.decodeFile(photoFile.absolutePath)
    }
}
cameraLauncher.launch(photoUri)
```

---

### Almacenamiento interno — archivos privados de la app

```kotlin
// Escribir en internal storage
fun writeInternalFile(context: Context, filename: String, content: String) {
    context.openFileOutput(filename, Context.MODE_PRIVATE).use { stream ->
        stream.write(content.toByteArray())
    }
}

// Leer desde internal storage
fun readInternalFile(context: Context, filename: String): String? {
    return try {
        context.openFileInput(filename).use { stream ->
            stream.bufferedReader().readText()
        }
    } catch (e: FileNotFoundException) { null }
}

// Listar archivos internos
val files = context.fileList()  // Array<String> de nombres

// Eliminar archivo interno
context.deleteFile("mi_archivo.txt")

// Acceso por File para operaciones más complejas
val file = File(context.filesDir, "subdirectorio/archivo.json")
file.parentFile?.mkdirs()  // Crear subdirectorios si no existen
file.writeText(jsonString)
val content = file.readText()

// Cache interno (el SO puede limpiar este directorio)
val cacheFile = File(context.cacheDir, "thumbnail_cache/img_123.jpg")
cacheFile.parentFile?.mkdirs()
```

---

### Almacenamiento externo privado (sin permisos en Android 10+)

```kotlin
// Directorio específico en sdcard/Android/data/<package>/files/
val picturesDir = context.getExternalFilesDir(Environment.DIRECTORY_PICTURES)
val downloadsDir = context.getExternalFilesDir(Environment.DIRECTORY_DOWNLOADS)
val documentsDir = context.getExternalFilesDir(Environment.DIRECTORY_DOCUMENTS)
val musicDir = context.getExternalFilesDir(Environment.DIRECTORY_MUSIC)
val moviesDir = context.getExternalFilesDir(Environment.DIRECTORY_MOVIES)

// Guardar imagen en directorio externo privado
val imageFile = File(picturesDir, "foto_perfil.jpg")
bitmap.compress(Bitmap.CompressFormat.JPEG, 90, imageFile.outputStream())

// Verificar disponibilidad del almacenamiento externo
val isAvailable = Environment.getExternalStorageState() == Environment.MEDIA_MOUNTED
val isReadOnly = Environment.getExternalStorageState() == Environment.MEDIA_MOUNTED_READ_ONLY

// Verificar espacio disponible
val statFs = StatFs(context.getExternalFilesDir(null)?.path ?: "")
val availableBytes = statFs.availableBlocksLong * statFs.blockSizeLong
val totalBytes = statFs.blockCountLong * statFs.blockSizeLong
```

---

### Acceso completo al almacenamiento (MANAGE_EXTERNAL_STORAGE)

Para casos excepcionales como administradores de archivos. Requiere justificación en Google Play.

```xml
<uses-permission android:name="android.permission.MANAGE_EXTERNAL_STORAGE"
    tools:ignore="ScopedStorage" />
```

```kotlin
// Verificar y solicitar
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R) {
    if (!Environment.isExternalStorageManager()) {
        val intent = Intent(Settings.ACTION_MANAGE_APP_ALL_FILES_ACCESS_PERMISSION).apply {
            data = Uri.parse("package:${context.packageName}")
        }
        startActivity(intent)
    } else {
        // Tiene acceso completo — puede usar File() con rutas absolutas
        val dcimDir = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DCIM)
        val files = dcimDir.listFiles()
    }
}
```

---

### MIME types comunes

| Tipo | MIME type |
|---|---|
| Imagen JPEG | `image/jpeg` |
| Imagen PNG | `image/png` |
| Imagen GIF | `image/gif` |
| Imagen WebP | `image/webp` |
| Cualquier imagen | `image/*` |
| Video MP4 | `video/mp4` |
| Cualquier video | `video/*` |
| Audio MP3 | `audio/mpeg` |
| Cualquier audio | `audio/*` |
| PDF | `application/pdf` |
| JSON | `application/json` |
| ZIP | `application/zip` |
| Texto plano | `text/plain` |
| CSV | `text/csv` |
| HTML | `text/html` |
| Cualquier archivo | `*/*` |

