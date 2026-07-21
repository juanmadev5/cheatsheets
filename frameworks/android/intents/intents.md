# Android Jetpack Compose — Intents

---

## 📨 Intents — Comunicación entre componentes

### Tipos de Intent

| Tipo | Descripción | Ejemplo |
|---|---|---|
| **Explícito** | Especifica el componente exacto (clase) a iniciar | Abrir tu propia Activity |
| **Implícito** | Declara una acción; el sistema elige qué app la maneja | Compartir texto, abrir URL |
| **Broadcast** | Mensaje de difusión a múltiples receptores | Batería baja, boot completado |
| **Pending Intent** | Intent diferido que otra app puede ejecutar en tu nombre | Notificaciones, alarmas, widgets |

---

### Intent explícito

```kotlin
// Abrir otra Activity de tu propia app
val intent = Intent(context, DetailActivity::class.java).apply {
    putExtra("product_id", 42)
    putExtra("product_name", "Laptop")
    putExtra("price", 999.99)
}
startActivity(intent)

// Leer extras en la Activity destino
val productId = intent.getIntExtra("product_id", -1)
val name = intent.getStringExtra("product_name") ?: ""
val price = intent.getDoubleExtra("price", 0.0)

// Pasar objetos Parcelable
@Parcelize
data class Product(val id: Int, val name: String) : Parcelable

val intent = Intent(context, DetailActivity::class.java).apply {
    putExtra("product", product)
}

val product = intent.getParcelableExtra<Product>("product")          // hasta API 32
val product = intent.getParcelableExtra("product", Product::class.java) // API 33+

// Iniciar Service explícito
val serviceIntent = Intent(context, SyncService::class.java).apply {
    action = "com.app.ACTION_SYNC"
    putExtra("force", true)
}
context.startService(serviceIntent)
context.startForegroundService(serviceIntent) // Para foreground services

// Iniciar con resultado (Activity Result API — moderno)
val launcher = rememberLauncherForActivityResult(
    ActivityResultContracts.StartActivityForResult()
) { result ->
    if (result.resultCode == Activity.RESULT_OK) {
        val data = result.data?.getStringExtra("result_key")
    }
}
launcher.launch(Intent(context, PickerActivity::class.java))

// Devolver resultado desde la Activity destino
setResult(Activity.RESULT_OK, Intent().apply { putExtra("result_key", "valor") })
finish()
```

---

### Intent implícito — acciones del sistema

```kotlin
// Abrir URL en el navegador
val intent = Intent(Intent.ACTION_VIEW, Uri.parse("https://developer.android.com"))
startActivity(intent)

// Abrir teléfono con número marcado (no llama — muestra el dialer)
val intent = Intent(Intent.ACTION_DIAL, Uri.parse("tel:+595981123456"))
startActivity(intent)

// Llamar directamente (requiere CALL_PHONE)
val intent = Intent(Intent.ACTION_CALL, Uri.parse("tel:+595981123456"))
startActivity(intent)

// Enviar email
val intent = Intent(Intent.ACTION_SENDTO).apply {
    data = Uri.parse("mailto:destino@email.com")
    putExtra(Intent.EXTRA_SUBJECT, "Asunto del email")
    putExtra(Intent.EXTRA_TEXT, "Cuerpo del mensaje")
}
startActivity(intent)

// Compartir texto (chooser — muestra selector de apps)
val intent = Intent(Intent.ACTION_SEND).apply {
    type = "text/plain"
    putExtra(Intent.EXTRA_TEXT, "Texto a compartir")
    putExtra(Intent.EXTRA_TITLE, "Título del share")
}
startActivity(Intent.createChooser(intent, "Compartir via..."))

// Compartir imagen
val intent = Intent(Intent.ACTION_SEND).apply {
    type = "image/jpeg"
    putExtra(Intent.EXTRA_STREAM, imageUri)
    addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION) // IMPORTANTE para URIs de FileProvider
}
startActivity(Intent.createChooser(intent, "Compartir imagen"))

// Compartir múltiples archivos
val intent = Intent(Intent.ACTION_SEND_MULTIPLE).apply {
    type = "image/*"
    putParcelableArrayListExtra(Intent.EXTRA_STREAM, ArrayList(uriList))
    addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION)
}
startActivity(Intent.createChooser(intent, "Compartir imágenes"))

// Abrir ajustes de la app
val intent = Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS).apply {
    data = Uri.parse("package:${context.packageName}")
}
startActivity(intent)

// Abrir ajustes de ubicación
startActivity(Intent(Settings.ACTION_LOCATION_SOURCE_SETTINGS))

// Abrir Google Maps con ubicación
val uri = Uri.parse("geo:${lat},${lng}?q=${lat},${lng}(Nombre del lugar)")
startActivity(Intent(Intent.ACTION_VIEW, uri))

// Abrir Google Maps con ruta
val uri = Uri.parse("https://maps.google.com/maps?daddr=${destLat},${destLng}")
startActivity(Intent(Intent.ACTION_VIEW, uri))

// Verificar si hay una app para manejar el Intent ANTES de lanzarlo
val packageManager = context.packageManager
if (intent.resolveActivity(packageManager) != null) {
    startActivity(intent)
} else {
    // No hay app disponible — mostrar mensaje al usuario
}
```

---

### Intent Flags importantes

```kotlin
val intent = Intent(context, HomeActivity::class.java).apply {

    // Limpiar el back stack y crear nueva tarea — útil para login/logout
    flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK

    // No crear nueva instancia si ya está en el back stack
    addFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP)

    // Llevar al frente si ya existe en alguna tarea
    addFlags(Intent.FLAG_ACTIVITY_REORDER_TO_FRONT)

    // Limpiar actividades encima hasta encontrar esta en el back stack
    addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP)

    // Otorgar permiso de lectura a la URI incluida (FileProvider)
    addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION)

    // Otorgar permiso de escritura a la URI incluida
    addFlags(Intent.FLAG_GRANT_WRITE_URI_PERMISSION)
}
```

---

### PendingIntent — para notificaciones, alarmas y widgets

```kotlin
// PendingIntent para abrir Activity desde una notificación
val intent = Intent(context, MainActivity::class.java).apply {
    flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK
    putExtra("notification_id", 123)
}
val pendingIntent = PendingIntent.getActivity(
    context,
    requestCode = 0,
    intent,
    PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE, // FLAG_IMMUTABLE requerido en Android 12+
)

// PendingIntent para Broadcast (botón en notificación)
val broadcastIntent = Intent(context, NotificationActionReceiver::class.java).apply {
    action = "com.app.ACTION_DISMISS"
    putExtra("notification_id", 123)
}
val broadcastPending = PendingIntent.getBroadcast(
    context,
    requestCode = 1,
    broadcastIntent,
    PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE,
)

// Usar en notificación
NotificationCompat.Builder(context, CHANNEL_ID)
    .setContentIntent(pendingIntent)          // Toque en la notificación
    .addAction(R.drawable.ic_close, "Cerrar", broadcastPending)  // Botón
    .build()

// PendingIntent para AlarmManager
val alarmManager = context.getSystemService(AlarmManager::class.java)
val triggerTime = System.currentTimeMillis() + TimeUnit.MINUTES.toMillis(30)

alarmManager.setExactAndAllowWhileIdle(
    AlarmManager.RTC_WAKEUP,
    triggerTime,
    pendingIntent,
)
```

---

### Deep Links — recibir Intents en tu app

```xml
<!-- AndroidManifest.xml — configurar intent-filter para deep links -->
<activity android:name=".MainActivity" android:exported="true">
    <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <!-- https://miapp.com/product/123 -->
        <data android:scheme="https" android:host="miapp.com" android:pathPrefix="/product" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <!-- miapp://product/123 -->
        <data android:scheme="miapp" android:host="product" />
    </intent-filter>
</activity>
```

```kotlin
// Leer el deep link en la Activity
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    handleIntent(intent)
}

override fun onNewIntent(intent: Intent) {
    super.onNewIntent(intent)
    handleIntent(intent)
}

private fun handleIntent(intent: Intent) {
    if (intent.action == Intent.ACTION_VIEW) {
        val uri = intent.data ?: return
        // https://miapp.com/product/123
        val productId = uri.lastPathSegment   // "123"
        val source = uri.getQueryParameter("source") // ?source=email
    }
}
```

