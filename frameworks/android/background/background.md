# Android Jetpack Compose — Background y servicios

---

## ⚙️ WorkManager

```kotlin
// Worker con Koin
class SyncWorker(
    context: Context,
    params: WorkerParameters,
    private val repository: ProductRepository,   // Inyectado por Koin
) : CoroutineWorker(context, params) {

    override suspend fun doWork(): Result {
        return try {
            val category = inputData.getString(KEY_CATEGORY) ?: ""
            repository.refreshProducts(category)

            // Output data para encadenar workers
            val outputData = workDataOf(KEY_COUNT to repository.getCount())
            Result.success(outputData)
        } catch (e: Exception) {
            if (runAttemptCount < 3) Result.retry() else Result.failure()
        }
    }

    companion object {
        const val KEY_CATEGORY = "category"
        const val KEY_COUNT = "count"
        const val WORK_NAME = "sync_products"
    }
}

// Módulo Koin para Workers
val workersModule = module {
    worker { SyncWorker(get(), get(), get()) }
}

// App.kt — registrar factory de Koin en WorkManager
startKoin {
    workManagerFactory()   // IMPORTANTE: conecta Koin con WorkManager
    modules(workersModule, /* ... */)
}

// Programar trabajo
class WorkScheduler(private val workManager: WorkManager) {

    // Una sola vez
    fun syncNow(category: String) {
        val inputData = workDataOf(SyncWorker.KEY_CATEGORY to category)
        val constraints = Constraints.Builder()
            .setRequiredNetworkType(NetworkType.CONNECTED)
            .setRequiresBatteryNotLow(true)
            .build()

        val request = OneTimeWorkRequestBuilder<SyncWorker>()
            .setInputData(inputData)
            .setConstraints(constraints)
            .setBackoffCriteria(BackoffPolicy.EXPONENTIAL, 30, TimeUnit.SECONDS)
            .addTag("sync")
            .build()

        workManager.enqueueUniqueWork(
            SyncWorker.WORK_NAME,
            ExistingWorkPolicy.REPLACE,
            request,
        )
    }

    // Periódico (mínimo 15 minutos)
    fun schedulePeriodicSync() {
        val request = PeriodicWorkRequestBuilder<SyncWorker>(1, TimeUnit.HOURS)
            .setConstraints(Constraints.Builder()
                .setRequiredNetworkType(NetworkType.CONNECTED)
                .build())
            .build()

        workManager.enqueueUniquePeriodicWork(
            "periodic_sync",
            ExistingPeriodicWorkPolicy.KEEP,
            request,
        )
    }

    // Encadenar workers
    fun chainedWork() {
        val sync = OneTimeWorkRequestBuilder<SyncWorker>().build()
        val cleanup = OneTimeWorkRequestBuilder<CleanupWorker>().build()
        val notify = OneTimeWorkRequestBuilder<NotifyWorker>().build()

        workManager
            .beginUniqueWork("chain", ExistingWorkPolicy.REPLACE, sync)
            .then(cleanup)
            .then(notify)
            .enqueue()
    }

    // Observar estado como Flow (integra directo con StateFlow/collectAsStateWithLifecycle)
    fun observeSync(): Flow<List<WorkInfo>> =
        workManager.getWorkInfosForUniqueWorkFlow(SyncWorker.WORK_NAME)

    // En el ViewModel
    // workRepository.observeSync()
    //     .onEach { workInfos -> workInfos.forEach { info ->
    //         when (info.state) {
    //             WorkInfo.State.SUCCEEDED -> { /* Completado */ }
    //             WorkInfo.State.FAILED -> { /* Falló */ }
    //             WorkInfo.State.RUNNING -> { /* En progreso */ }
    //             else -> Unit
    //         }
    //     } }
    //     .launchIn(viewModelScope)
}
```

---

## 🔧 Foreground Service

```kotlin
// MusicPlayerService.kt
class MusicPlayerService : Service() {

    private val binder = LocalBinder()
    inner class LocalBinder : Binder() {
        fun getService(): MusicPlayerService = this@MusicPlayerService
    }

    override fun onBind(intent: Intent): IBinder = binder

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        when (intent?.action) {
            ACTION_PLAY -> play(intent.getStringExtra("track_url"))
            ACTION_PAUSE -> pause()
            ACTION_STOP -> stopSelf()
        }
        return START_NOT_STICKY  // No recrear si el sistema lo mata
        // START_STICKY          → recrear con intent null
        // START_REDELIVER_INTENT → recrear con último intent
    }

    private fun play(url: String?) {
        val notification = buildNotification()
        // Android 14+ requiere declarar el tipo de foreground service
        ServiceCompat.startForeground(
            this,
            NOTIFICATION_ID,
            notification,
            ServiceInfo.FOREGROUND_SERVICE_TYPE_MEDIA_PLAYBACK,
        )
        // ... lógica de reproducción ...
    }

    private fun pause() { /* pausar media player */ }

    private fun buildNotification(): Notification {
        val stopPending = PendingIntent.getService(
            this, 0,
            Intent(this, MusicPlayerService::class.java).apply { action = ACTION_STOP },
            PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE,
        )
        return NotificationCompat.Builder(this, CHANNEL_ID)
            .setSmallIcon(R.drawable.ic_music)
            .setContentTitle("Reproduciendo")
            .setContentText("Nombre de la canción")
            .addAction(R.drawable.ic_stop, "Detener", stopPending)
            .setOngoing(true)        // No puede ser descartada con swipe
            .setSilent(true)
            .build()
    }

    override fun onDestroy() {
        super.onDestroy()
        ServiceCompat.stopForeground(this, ServiceCompat.STOP_FOREGROUND_REMOVE)
    }

    companion object {
        const val ACTION_PLAY = "com.app.PLAY"
        const val ACTION_PAUSE = "com.app.PAUSE"
        const val ACTION_STOP = "com.app.STOP"
        const val NOTIFICATION_ID = 101
        const val CHANNEL_ID = "media_playback"
    }
}
```

```xml
<!-- AndroidManifest.xml -->
<service
    android:name=".MusicPlayerService"
    android:exported="false"
    android:foregroundServiceType="mediaPlayback" />
```

```kotlin
// Iniciar el service desde Composable/Activity
val context = LocalContext.current

// Iniciar
Intent(context, MusicPlayerService::class.java).also { intent ->
    intent.action = MusicPlayerService.ACTION_PLAY
    intent.putExtra("track_url", "https://...")
    context.startForegroundService(intent)
}

// Bindear para comunicación bidireccional
var service: MusicPlayerService? = null
val connection = object : ServiceConnection {
    override fun onServiceConnected(name: ComponentName, binder: IBinder) {
        service = (binder as MusicPlayerService.LocalBinder).getService()
    }
    override fun onServiceDisconnected(name: ComponentName) { service = null }
}
context.bindService(
    Intent(context, MusicPlayerService::class.java),
    connection,
    Context.BIND_AUTO_CREATE,
)
// No olvidar unbindService(connection) en onStop/onDestroy
```

---

## 📡 Broadcast Receivers

```kotlin
// BroadcastReceiver básico
class NetworkReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        when (intent.action) {
            ConnectivityManager.CONNECTIVITY_ACTION -> {
                val connected = isConnected(context)
                // Notificar a la app
            }
            Intent.ACTION_BOOT_COMPLETED -> {
                // Re-programar alarmas o workers
            }
        }
    }

    private fun isConnected(context: Context): Boolean {
        val cm = context.getSystemService(Context.CONNECTIVITY_SERVICE) as ConnectivityManager
        return cm.activeNetwork?.let {
            cm.getNetworkCapabilities(it)?.hasCapability(NetworkCapabilities.NET_CAPABILITY_INTERNET)
        } ?: false
    }
}

// Registrar en AndroidManifest.xml (para acciones del sistema al iniciar)
// <receiver android:name=".NetworkReceiver" android:exported="false">
//     <intent-filter>
//         <action android:name="android.intent.action.BOOT_COMPLETED"/>
//     </intent-filter>
// </receiver>

// Registrar dinámicamente (recomendado para la mayoría de casos)
class MainActivity : ComponentActivity() {
    private val networkReceiver = NetworkReceiver()

    override fun onStart() {
        super.onStart()
        val filter = IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION)
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            registerReceiver(networkReceiver, filter, RECEIVER_NOT_EXPORTED)
        } else {
            registerReceiver(networkReceiver, filter)
        }
    }

    override fun onStop() {
        super.onStop()
        unregisterReceiver(networkReceiver)
    }
}

// Receiver con Flow para consumir en ViewModel (moderno)
fun Context.networkStatusFlow(): Flow<Boolean> = callbackFlow {
    val receiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context, intent: Intent) {
            trySend(isNetworkAvailable(context))
        }
    }
    val filter = IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION)
    registerReceiver(receiver, filter)
    trySend(isNetworkAvailable(this@networkStatusFlow))  // Emitir estado inicial

    awaitClose { unregisterReceiver(receiver) }
}.distinctUntilChanged()

// Broadcast local (entre componentes de la misma app)
val localBroadcastManager = LocalBroadcastManager.getInstance(context)

// Enviar
val intent = Intent("com.app.ACTION_REFRESH").apply { putExtra("key", "value") }
localBroadcastManager.sendBroadcast(intent)

// Recibir
val receiver = object : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) { /* ... */ }
}
localBroadcastManager.registerReceiver(receiver, IntentFilter("com.app.ACTION_REFRESH"))
localBroadcastManager.unregisterReceiver(receiver)
```

