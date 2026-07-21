# Android Jetpack Compose — Connectivity

---

## 📶 Connectivity — monitoreo de red

```kotlin
// NetworkMonitor.kt — observar conectividad como Flow
class NetworkMonitor(private val context: Context) {

    val isOnline: Flow<Boolean> = callbackFlow {
        val connectivityManager = context.getSystemService(ConnectivityManager::class.java)

        // Emitir estado inicial
        trySend(connectivityManager.isCurrentlyConnected())

        val callback = object : ConnectivityManager.NetworkCallback() {
            override fun onAvailable(network: Network) { trySend(true) }
            override fun onLost(network: Network) { trySend(false) }
            override fun onCapabilitiesChanged(network: Network, caps: NetworkCapabilities) {
                trySend(caps.hasCapability(NetworkCapabilities.NET_CAPABILITY_INTERNET) &&
                        caps.hasCapability(NetworkCapabilities.NET_CAPABILITY_VALIDATED))
            }
        }

        val request = NetworkRequest.Builder()
            .addCapability(NetworkCapabilities.NET_CAPABILITY_INTERNET)
            .addTransportType(NetworkCapabilities.TRANSPORT_WIFI)
            .addTransportType(NetworkCapabilities.TRANSPORT_CELLULAR)
            .build()

        connectivityManager.registerNetworkCallback(request, callback)
        awaitClose { connectivityManager.unregisterNetworkCallback(callback) }
    }.distinctUntilChanged()

    private fun ConnectivityManager.isCurrentlyConnected(): Boolean {
        val network = activeNetwork ?: return false
        val caps = getNetworkCapabilities(network) ?: return false
        return caps.hasCapability(NetworkCapabilities.NET_CAPABILITY_INTERNET) &&
               caps.hasCapability(NetworkCapabilities.NET_CAPABILITY_VALIDATED)
    }
}

// Módulo Koin
val networkModule = module {
    single { NetworkMonitor(get()) }
}

// ViewModel con retry automático al volver online
class ProductsViewModel(
    private val repo: ProductRepository,
    private val networkMonitor: NetworkMonitor,
) : ViewModel() {

    init {
        // Recargar datos cuando vuelve la conexión
        viewModelScope.launch {
            networkMonitor.isOnline
                .filter { it }                        // Solo cuando vuelve a estar online
                .distinctUntilChanged()
                .drop(1)                              // Ignorar el estado inicial
                .collect { loadProducts() }
        }
    }
}

// Banner de sin conexión en Composable
@Composable
fun OfflineBanner() {
    val networkMonitor: NetworkMonitor = koinInject()
    val isOnline by networkMonitor.isOnline.collectAsStateWithLifecycle(initialValue = true)

    AnimatedVisibility(
        visible = !isOnline,
        enter = slideInVertically() + fadeIn(),
        exit = slideOutVertically() + fadeOut(),
    ) {
        Surface(color = MaterialTheme.colorScheme.errorContainer) {
            Row(
                modifier = Modifier.fillMaxWidth().padding(8.dp),
                horizontalArrangement = Arrangement.Center,
                verticalAlignment = Alignment.CenterVertically,
            ) {
                Icon(Icons.Default.WifiOff, contentDescription = null,
                    tint = MaterialTheme.colorScheme.onErrorContainer)
                Spacer(Modifier.width(8.dp))
                Text("Sin conexión a internet",
                    color = MaterialTheme.colorScheme.onErrorContainer)
            }
        }
    }
}
```

