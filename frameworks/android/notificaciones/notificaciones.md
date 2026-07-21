# Android Jetpack Compose — Notificaciones

---

## 🔔 Notificaciones

```kotlin
// NotificationHelper.kt
class NotificationHelper(private val context: Context) {

    companion object {
        const val CHANNEL_GENERAL = "channel_general"
        const val CHANNEL_SYNC = "channel_sync"
    }

    fun createChannels() {
        val channels = listOf(
            NotificationChannel(CHANNEL_GENERAL, "General", NotificationManager.IMPORTANCE_DEFAULT)
                .apply { description = "Notificaciones generales" },
            NotificationChannel(CHANNEL_SYNC, "Sincronización", NotificationManager.IMPORTANCE_LOW)
                .apply { description = "Estado de sincronización" },
        )
        val manager = context.getSystemService(NotificationManager::class.java)
        channels.forEach { manager.createNotificationChannel(it) }
    }

    fun showNotification(title: String, body: String, id: Int = 1) {
        val intent = Intent(context, MainActivity::class.java).apply {
            flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK
        }
        val pendingIntent = PendingIntent.getActivity(
            context, 0, intent,
            PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE,
        )

        val notification = NotificationCompat.Builder(context, CHANNEL_GENERAL)
            .setSmallIcon(R.drawable.ic_notification)
            .setContentTitle(title)
            .setContentText(body)
            .setStyle(NotificationCompat.BigTextStyle().bigText(body))
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)
            .setContentIntent(pendingIntent)
            .setAutoCancel(true)
            .build()

        NotificationManagerCompat.from(context).notify(id, notification)
    }
}

// App.kt — crear canales al iniciar
class App : Application() {
    override fun onCreate() {
        super.onCreate()
        NotificationHelper(this).createChannels()
        // ...
    }
}
```

