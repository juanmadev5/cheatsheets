# Android Jetpack Compose — Play Store

---

## 🏪 In-App Updates e In-App Reviews

```kotlin
// build.gradle.kts
// implementation("com.google.android.play:app-update-ktx:2.1.0")
// implementation("com.google.android.play:review-ktx:2.0.2")

// In-App Updates
class UpdateManager(private val activity: Activity) {

    private val appUpdateManager = AppUpdateManagerFactory.create(activity)

    // Verificar si hay actualización disponible
    suspend fun checkForUpdate(): AppUpdateInfo? = suspendCancellableCoroutine { cont ->
        appUpdateManager.appUpdateInfo.addOnSuccessListener { info ->
            if (info.updateAvailability() == UpdateAvailability.UPDATE_AVAILABLE) {
                cont.resume(info)
            } else {
                cont.resume(null)
            }
        }.addOnFailureListener { cont.resume(null) }
    }

    // Actualización flexible (en background — recomendada para updates no críticos)
    fun startFlexibleUpdate(info: AppUpdateInfo, launcher: ActivityResultLauncher<IntentSenderRequest>) {
        appUpdateManager.startUpdateFlowForResult(
            info,
            launcher,
            AppUpdateOptions.newBuilder(AppUpdateType.FLEXIBLE).build(),
        )
    }

    // Actualización inmediata (bloquea la app — para updates críticos de seguridad)
    fun startImmediateUpdate(info: AppUpdateInfo, launcher: ActivityResultLauncher<IntentSenderRequest>) {
        appUpdateManager.startUpdateFlowForResult(
            info,
            launcher,
            AppUpdateOptions.newBuilder(AppUpdateType.IMMEDIATE).build(),
        )
    }

    // Completar actualización flexible (instalar después de descargada)
    fun completeFlexibleUpdate() = appUpdateManager.completeUpdate()

    fun registerInstallStateListener(onDownloaded: () -> Unit): InstallStateUpdatedListener {
        val listener = InstallStateUpdatedListener { state ->
            if (state.installStatus() == InstallStatus.DOWNLOADED) onDownloaded()
        }
        appUpdateManager.registerListener(listener)
        return listener
    }

    fun unregisterListener(listener: InstallStateUpdatedListener) =
        appUpdateManager.unregisterListener(listener)
}

// In-App Review
class ReviewManager(private val context: Context) {
    private val reviewManager = ReviewManagerFactory.create(context)

    suspend fun requestReview(activity: Activity) {
        try {
            val reviewInfo = reviewManager.requestReviewFlow().await()
            reviewManager.launchReviewFlow(activity, reviewInfo).await()
            // Nota: Google no garantiza que el dialog se muestre siempre
            // (tiene límites internos de frecuencia)
        } catch (e: Exception) {
            // Falla silenciosamente — nunca mostrar error al usuario por esto
        }
    }
}

// Usar en Composable/ViewModel
@Composable
fun AppUpdateHandler() {
    val context = LocalContext.current
    val activity = context as Activity
    val updateManager = remember { UpdateManager(activity) }

    val updateLauncher = rememberLauncherForActivityResult(
        ActivityResultContracts.StartIntentSenderForResult()
    ) { result ->
        if (result.resultCode == Activity.RESULT_CANCELED) {
            // Usuario canceló — manejar según criticidad
        }
    }

    LaunchedEffect(Unit) {
        val updateInfo = updateManager.checkForUpdate() ?: return@LaunchedEffect
        val priority = updateInfo.updatePriority()

        if (priority >= 4) {
            // Alta prioridad → actualización inmediata
            updateManager.startImmediateUpdate(updateInfo, updateLauncher)
        } else {
            // Baja prioridad → flexible
            updateManager.startFlexibleUpdate(updateInfo, updateLauncher)
        }
    }
}
```

