# Android Jetpack Compose — Permisos y Manifest

---

## 📋 AndroidManifest.xml — Permisos completos

### Estructura básica del Manifest

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <!-- Permisos que necesita la app -->
    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:name=".App"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/Theme.MyApp"
        android:usesCleartextTraffic="false">

        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:windowSoftInputMode="adjustResize">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

    </application>
</manifest>
```

---

### Tipos de permisos según su nivel de protección

| Nivel | Descripción | Se necesita pedir en runtime? |
|---|---|---|
| `normal` | Riesgo bajo, se otorga automáticamente al instalar | No |
| `dangerous` | Accede a datos sensibles del usuario | **Sí** (Android 6+) |
| `signature` | Solo apps firmadas con el mismo certificado | No |
| `signatureOrSystem` | Apps del sistema o misma firma | No |

---

### 🌐 Red y conectividad

```xml
<!-- Acceso a Internet — normal -->
<uses-permission android:name="android.permission.INTERNET" />

<!-- Ver estado de la red (WiFi/datos) — normal -->
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

<!-- Ver estado del WiFi — normal -->
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />

<!-- Cambiar conectividad WiFi — normal -->
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />

<!-- Cambiar conectividad de red — normal -->
<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />

<!-- Bluetooth (descubrir/conectar) — dangerous (Android 12+) -->
<uses-permission android:name="android.permission.BLUETOOTH_SCAN"
    android:usesPermissionFlags="neverForLocation" />
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
<uses-permission android:name="android.permission.BLUETOOTH_ADVERTISE" />

<!-- NFC — normal -->
<uses-permission android:name="android.permission.NFC" />
```

---

### 📍 Ubicación

```xml
<!-- Ubicación aproximada (ciudad/barrio) — dangerous -->
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />

<!-- Ubicación precisa (GPS) — dangerous -->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />

<!-- Ubicación en background (siempre, incluso con app cerrada) — dangerous -->
<!-- REQUIERE pedir ACCESS_FINE/COARSE primero, luego pedir este por separado -->
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
```

> ⚠️ En Android 10+ para ubicación en background el usuario debe ir manualmente a Ajustes y seleccionar "Permitir siempre".

---

### 📷 Cámara y multimedia

```xml
<!-- Cámara — dangerous -->
<uses-permission android:name="android.permission.CAMERA" />

<!-- Micrófono (grabar audio) — dangerous -->
<uses-permission android:name="android.permission.RECORD_AUDIO" />

<!-- Leer imágenes (Android 13+) — dangerous -->
<uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />

<!-- Leer videos (Android 13+) — dangerous -->
<uses-permission android:name="android.permission.READ_MEDIA_VIDEO" />

<!-- Leer audio (Android 13+) — dangerous -->
<uses-permission android:name="android.permission.READ_MEDIA_AUDIO" />

<!-- Leer almacenamiento externo (hasta Android 12) — dangerous -->
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"
    android:maxSdkVersion="32" />

<!-- Escribir en almacenamiento externo (hasta Android 9) — dangerous -->
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
    android:maxSdkVersion="28" />

<!-- Selección de fotos parcial (Android 14+) — dangerous -->
<uses-permission android:name="android.permission.READ_MEDIA_VISUAL_USER_SELECTED" />
```

---

### 👤 Contactos y calendarios

```xml
<!-- Leer contactos — dangerous -->
<uses-permission android:name="android.permission.READ_CONTACTS" />

<!-- Escribir/modificar contactos — dangerous -->
<uses-permission android:name="android.permission.WRITE_CONTACTS" />

<!-- Obtener lista de cuentas del dispositivo — dangerous -->
<uses-permission android:name="android.permission.GET_ACCOUNTS" />

<!-- Leer eventos del calendario — dangerous -->
<uses-permission android:name="android.permission.READ_CALENDAR" />

<!-- Crear/modificar eventos del calendario — dangerous -->
<uses-permission android:name="android.permission.WRITE_CALENDAR" />
```

---

### 📞 Teléfono y SMS

```xml
<!-- Hacer llamadas — dangerous -->
<uses-permission android:name="android.permission.CALL_PHONE" />

<!-- Leer estado del teléfono (número, IMEI) — dangerous -->
<uses-permission android:name="android.permission.READ_PHONE_STATE" />

<!-- Leer números de teléfono — dangerous -->
<uses-permission android:name="android.permission.READ_PHONE_NUMBERS" />

<!-- Responder llamadas — dangerous -->
<uses-permission android:name="android.permission.ANSWER_PHONE_CALLS" />

<!-- Leer SMS — dangerous -->
<uses-permission android:name="android.permission.READ_SMS" />

<!-- Enviar SMS — dangerous -->
<uses-permission android:name="android.permission.SEND_SMS" />

<!-- Recibir SMS — dangerous -->
<uses-permission android:name="android.permission.RECEIVE_SMS" />

<!-- Leer log de llamadas — dangerous -->
<uses-permission android:name="android.permission.READ_CALL_LOG" />
```

---

### 🔔 Notificaciones y alarmas

```xml
<!-- Mostrar notificaciones (Android 13+) — dangerous -->
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />

<!-- Programar alarmas exactas (Android 12+) — special -->
<uses-permission android:name="android.permission.SCHEDULE_EXACT_ALARM" />

<!-- Alarmas exactas con permiso especial (Android 13+) — special -->
<uses-permission android:name="android.permission.USE_EXACT_ALARM" />
```

---

### 🔋 Batería y background

```xml
<!-- Ignorar optimización de batería (correr en background) — special -->
<!-- El usuario debe habilitarlo manualmente desde Ajustes -->
<uses-permission android:name="android.permission.REQUEST_IGNORE_BATTERY_OPTIMIZATIONS" />

<!-- Mantener CPU activa con WakeLock — normal -->
<uses-permission android:name="android.permission.WAKE_LOCK" />

<!-- Arrancar al iniciar el dispositivo — normal -->
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />

<!-- Servicio en primer plano — normal -->
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />

<!-- Tipo específico de foreground service (Android 14+) -->
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_CAMERA" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_LOCATION" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_MICROPHONE" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_MEDIA_PLAYBACK" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_DATA_SYNC" />
```

---

### 🏃 Sensores y actividad física

```xml
<!-- Reconocimiento de actividad física (pasos, etc.) — dangerous (Android 10+) -->
<uses-permission android:name="android.permission.ACTIVITY_RECOGNITION" />

<!-- Sensor de frecuencia cardíaca (Wear OS) — dangerous -->
<uses-permission android:name="android.permission.BODY_SENSORS" />

<!-- Sensor de frecuencia en background (Wear OS) — dangerous -->
<uses-permission android:name="android.permission.BODY_SENSORS_BACKGROUND" />
```

---

### 🔒 Permisos especiales (requieren Intent a Ajustes)

Estos permisos no se piden con `requestPermissions()` sino redirigiendo al usuario a la pantalla de ajustes:

```xml
<!-- Instalar apps de fuentes desconocidas -->
<uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES" />

<!-- Dibujar sobre otras apps (overlay) -->
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />

<!-- Acceso a configuración del sistema -->
<uses-permission android:name="android.permission.WRITE_SETTINGS" />

<!-- Acceso a notificaciones de otras apps -->
<uses-permission android:name="android.permission.BIND_NOTIFICATION_LISTENER_SERVICE" />

<!-- Gestor de contraseñas / AutoFill -->
<uses-permission android:name="android.permission.BIND_AUTOFILL_SERVICE" />
```

```kotlin
// Verificar y redirigir a ajustes para permisos especiales
if (!Settings.canDrawOverlays(context)) {
    val intent = Intent(
        Settings.ACTION_MANAGE_OVERLAY_PERMISSION,
        Uri.parse("package:${context.packageName}")
    )
    startActivity(intent)
}

if (!Environment.isExternalStorageManager()) {
    val intent = Intent(Settings.ACTION_MANAGE_APP_ALL_FILES_ACCESS_PERMISSION)
    startActivity(intent)
}
```

---

### 📲 Pedir permisos en runtime (Compose)

```kotlin
// Permiso único
val launcher = rememberLauncherForActivityResult(
    ActivityResultContracts.RequestPermission()
) { isGranted ->
    if (isGranted) { /* usar la funcionalidad */ }
    else { /* mostrar rationale o deshabilitar feature */ }
}

Button(onClick = { launcher.launch(Manifest.permission.CAMERA) }) {
    Text("Abrir cámara")
}

// Múltiples permisos a la vez
val multiplePermissionsLauncher = rememberLauncherForActivityResult(
    ActivityResultContracts.RequestMultiplePermissions()
) { permissions ->
    val cameraGranted = permissions[Manifest.permission.CAMERA] ?: false
    val micGranted = permissions[Manifest.permission.RECORD_AUDIO] ?: false
    if (cameraGranted && micGranted) { /* iniciar videollamada */ }
}

Button(onClick = {
    multiplePermissionsLauncher.launch(
        arrayOf(Manifest.permission.CAMERA, Manifest.permission.RECORD_AUDIO)
    )
}) {
    Text("Iniciar videollamada")
}

// Verificar si ya tiene el permiso
val hasPermission = ContextCompat.checkSelfPermission(
    context,
    Manifest.permission.CAMERA
) == PackageManager.PERMISSION_GRANTED

// Verificar si mostrar rationale (usuario denegó previamente sin "no preguntar más")
val shouldShowRationale = ActivityCompat.shouldShowRequestPermissionRationale(
    activity,
    Manifest.permission.CAMERA
)
```

---

### 📦 `uses-feature` — declarar hardware requerido

```xml
<!-- La app requiere cámara (excluye del Play Store dispositivos sin cámara) -->
<uses-feature android:name="android.hardware.camera" android:required="true" />

<!-- Cámara trasera -->
<uses-feature android:name="android.hardware.camera.any" android:required="false" />

<!-- GPS -->
<uses-feature android:name="android.hardware.location.gps" android:required="false" />

<!-- Bluetooth -->
<uses-feature android:name="android.hardware.bluetooth" android:required="false" />

<!-- Pantalla táctil -->
<uses-feature android:name="android.hardware.touchscreen" android:required="false" />

<!-- NFC -->
<uses-feature android:name="android.hardware.nfc" android:required="false" />

<!-- Sensor de huellas -->
<uses-feature android:name="android.hardware.fingerprint" android:required="false" />
```

> Usar `android:required="false"` para features opcionales — así la app está disponible en más dispositivos y vos verificás en código si el hardware existe.

