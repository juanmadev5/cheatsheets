# Android Jetpack Compose — Temas

---

## 🎨 Temas con Material 3

```kotlin
// ui/theme/Color.kt
val PrimaryBlue = Color(0xFF2563EB)
val OnPrimaryBlue = Color(0xFFFFFFFF)
val SurfaceLight = Color(0xFFF8FAFC)
val SurfaceDark = Color(0xFF1E293B)

// ui/theme/Theme.kt
private val LightColorScheme = lightColorScheme(
    primary = PrimaryBlue,
    onPrimary = OnPrimaryBlue,
    surface = SurfaceLight,
)

private val DarkColorScheme = darkColorScheme(
    primary = Color(0xFF93C5FD),
    surface = SurfaceDark,
)

@Composable
fun MyAppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = true,     // Material You (Android 12+)
    content: @Composable () -> Unit,
) {
    val colorScheme = when {
        dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            val context = LocalContext.current
            if (darkTheme) dynamicDarkColorScheme(context) else dynamicLightColorScheme(context)
        }
        darkTheme -> DarkColorScheme
        else -> LightColorScheme
    }

    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography,
        content = content,
    )
}

// Acceder al tema en composables
val colors = MaterialTheme.colorScheme
val typography = MaterialTheme.typography
val shapes = MaterialTheme.shapes

Text("Hola", style = typography.headlineMedium, color = colors.primary)
Surface(color = colors.surfaceVariant, shape = shapes.medium) { /* ... */ }
```

---

## 🚀 Splash Screen API

```kotlin
// build.gradle.kts
// implementation("androidx.core:core-splashscreen:1.2.0")

// MainActivity.kt
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        // installSplashScreen() SIEMPRE antes de super.onCreate()
        val splashScreen = installSplashScreen()
        super.onCreate(savedInstanceState)

        // Mantener la splash visible hasta que algo esté listo (ej: sesión ya cargada)
        val viewModel: MainViewModel by viewModels()
        splashScreen.setKeepOnScreenCondition { !viewModel.isReady.value }

        enableEdgeToEdge()
        setContent {
            MyAppTheme { AppNavGraph() }
        }
    }
}
```

```xml
<!-- res/values/themes.xml -->
<style name="Theme.App.Splash" parent="Theme.SplashScreen">
    <item name="windowSplashScreenBackground">@color/splash_background</item>
    <!-- Debe ser un AnimatedVectorDrawable, duración recomendada ≤ 1000ms -->
    <item name="windowSplashScreenAnimatedIcon">@drawable/splash_icon_animated</item>
    <item name="windowSplashScreenIconBackgroundColor">@color/splash_icon_background</item>
    <!-- Tema que se aplica apenas termina de mostrarse la splash -->
    <item name="postSplashScreenTheme">@style/Theme.MyApp</item>
</style>
```

```xml
<!-- AndroidManifest.xml — usar el tema de splash como tema inicial de la Activity -->
<application android:theme="@style/Theme.App.Splash">
    <activity android:name=".MainActivity" android:exported="true" android:theme="@style/Theme.App.Splash">
        ...
    </activity>
</application>
```

> El ícono animado y el color de fondo se dibujan por el propio sistema **antes** de que el proceso de la app termine de arrancar — no depende de Compose ni corre código tuyo hasta después. `setKeepOnScreenCondition` es lo único que controla cuánto dura.

