# Android Jetpack Compose — Seguridad

---

## 🔒 Seguridad

### Almacenamiento seguro — DataStore + Tink + Android Keystore

> `EncryptedSharedPreferences` está deprecada desde `androidx.security:security-crypto:1.1.0` (por problemas de performance en el hilo principal y corrupción de keysets en algunos OEM). El reemplazo recomendado es cifrar los valores con [Tink](https://github.com/tink-crypto/tink) — usando una clave protegida por el Android Keystore — y persistirlos en DataStore.

```kotlin
// build.gradle.kts
// implementation("com.google.crypto.tink:tink-android:1.19.0")
// implementation("androidx.datastore:datastore-preferences:1.2.1")

// SecureStorage.kt
private val Context.secureDataStore by preferencesDataStore(name = "secure_prefs")

class SecureStorage(private val context: Context) {

    private val aead: Aead by lazy {
        AeadConfig.register()
        AndroidKeysetManager.Builder()
            .withSharedPref(context, "master_keyset", "master_keyset_prefs")
            .withKeyTemplate(AesGcmKeyManager.aes256GcmTemplate())
            .withMasterKeyUri("android-keystore://secure_storage_master_key")
            .build()
            .keysetHandle
            .getPrimitive(Aead::class.java)
    }

    companion object {
        private val AUTH_TOKEN = stringPreferencesKey("auth_token")
    }

    private fun encrypt(plainText: String): String =
        Base64.encodeToString(aead.encrypt(plainText.toByteArray(), null), Base64.NO_WRAP)

    private fun decrypt(cipherText: String): String =
        String(aead.decrypt(Base64.decode(cipherText, Base64.NO_WRAP), null))

    suspend fun saveAuthToken(token: String) {
        context.secureDataStore.edit { prefs -> prefs[AUTH_TOKEN] = encrypt(token) }
    }

    val authToken: Flow<String?> = context.secureDataStore.data
        .map { prefs -> prefs[AUTH_TOKEN]?.let { decrypt(it) } }
}
```

---

### BiometricPrompt — huella digital y reconocimiento facial

```kotlin
// build.gradle.kts
// implementation("androidx.biometric:biometric:1.1.0")

class BiometricAuthManager(private val activity: FragmentActivity) {

    fun isBiometricAvailable(): Boolean {
        val biometricManager = BiometricManager.from(activity)
        return biometricManager.canAuthenticate(
            BiometricManager.Authenticators.BIOMETRIC_STRONG or
            BiometricManager.Authenticators.DEVICE_CREDENTIAL
        ) == BiometricManager.BIOMETRIC_SUCCESS
    }

    fun authenticate(
        title: String = "Autenticación requerida",
        subtitle: String = "Usá tu huella o PIN para continuar",
        onSuccess: () -> Unit,
        onError: (String) -> Unit,
        onFailed: () -> Unit = {},
    ) {
        val executor = ContextCompat.getMainExecutor(activity)
        val callback = object : BiometricPrompt.AuthenticationCallback() {
            override fun onAuthenticationSucceeded(result: BiometricPrompt.AuthenticationResult) {
                onSuccess()
            }
            override fun onAuthenticationError(errorCode: Int, errString: CharSequence) {
                if (errorCode != BiometricPrompt.ERROR_USER_CANCELED &&
                    errorCode != BiometricPrompt.ERROR_NEGATIVE_BUTTON) {
                    onError(errString.toString())
                }
            }
            override fun onAuthenticationFailed() = onFailed()
        }

        val promptInfo = BiometricPrompt.PromptInfo.Builder()
            .setTitle(title)
            .setSubtitle(subtitle)
            .setAllowedAuthenticators(
                BiometricManager.Authenticators.BIOMETRIC_STRONG or
                BiometricManager.Authenticators.DEVICE_CREDENTIAL
            )
            .build()

        BiometricPrompt(activity, executor, callback).authenticate(promptInfo)
    }
}

// Uso en Composable
@Composable
fun BiometricLoginButton() {
    val activity = LocalContext.current as FragmentActivity
    val biometricManager = remember { BiometricAuthManager(activity) }

    if (biometricManager.isBiometricAvailable()) {
        Button(onClick = {
            biometricManager.authenticate(
                onSuccess = { /* proceder */ },
                onError = { message -> /* mostrar error */ },
            )
        }) {
            Icon(Icons.Default.Fingerprint, contentDescription = null)
            Spacer(Modifier.width(8.dp))
            Text("Ingresar con huella")
        }
    }
}
```

---

### Network Security Config — HTTPS y certificate pinning

```xml
<!-- res/xml/network_security_config.xml -->
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <!-- Permitir tráfico HTTP solo en debug hacia emulador -->
    <domain-config cleartextTrafficPermitted="false">
        <domain includeSubdomains="true">api.miapp.com</domain>
        <pin-set expiration="2028-12-31">
            <pin digest="SHA-256">AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=</pin>
            <pin digest="SHA-256">BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=</pin>
        </pin-set>
    </domain-config>
    <debug-overrides>
        <trust-anchors>
            <certificates src="user" />  <!-- Confiar en certs del usuario en debug (Charles/Proxyman) -->
        </trust-anchors>
    </debug-overrides>
</network-security-config>
```

```xml
<!-- AndroidManifest.xml -->
<application
    android:networkSecurityConfig="@xml/network_security_config"
    android:usesCleartextTraffic="false">
```

