# Android Jetpack Compose — Base

---

## ⚙️ Gradle — dependencias esenciales

```kotlin
// build.gradle.kts (app)
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.compose)
    alias(libs.plugins.ksp)                    // KSP para Room, Koin annotations
    alias(libs.plugins.kotlin.serialization)
}

android {
    compileSdk = 36
    defaultConfig {
        minSdk = 26
        targetSdk = 36
    }
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
    kotlinOptions { jvmTarget = "17" }
    buildFeatures { compose = true }
}

dependencies {
    // Compose BOM — versiona todo Compose automáticamente
    val composeBom = platform("androidx.compose:compose-bom:2026.06.01")
    implementation(composeBom)
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.ui:ui-tooling-preview")
    implementation("androidx.compose.material3:material3")
    debugImplementation("androidx.compose.ui:ui-tooling")

    // Activity & Lifecycle
    implementation("androidx.activity:activity-compose:1.13.0")
    implementation("androidx.lifecycle:lifecycle-runtime-compose:2.11.0")
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.11.0")

    // Navigation 3
    implementation("androidx.navigation3:navigation3-runtime:1.1.4")
    implementation("androidx.navigation3:navigation3-ui:1.1.4")
    implementation("androidx.lifecycle:lifecycle-viewmodel-navigation3:2.11.0")

    // Room
    implementation("androidx.room:room-runtime:2.8.4")
    implementation("androidx.room:room-ktx:2.8.4")
    ksp("androidx.room:room-compiler:2.8.4")

    // DataStore
    implementation("androidx.datastore:datastore-preferences:1.2.1")

    // WorkManager
    implementation("androidx.work:work-runtime-ktx:2.11.2")

    // Paging 3
    implementation("androidx.paging:paging-runtime-ktx:3.5.0")
    implementation("androidx.paging:paging-compose:3.5.0")

    // Koin — BOM recomendado para mantener todas las versiones alineadas
    implementation(platform("io.insert-koin:koin-bom:4.1.1"))
    implementation("io.insert-koin:koin-android")
    implementation("io.insert-koin:koin-androidx-compose")
    implementation("io.insert-koin:koin-androidx-workmanager")

    // Retrofit + OkHttp
    implementation("com.squareup.retrofit2:retrofit:3.0.0")
    implementation("com.squareup.retrofit2:converter-kotlinx-serialization:3.0.0")
    implementation("com.squareup.okhttp3:logging-interceptor:5.4.0")

    // Kotlin Serialization
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.11.0")

    // Coroutines
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.11.0")

    // Coil — imágenes
    implementation("io.coil-kt.coil3:coil-compose:3.5.0")
    implementation("io.coil-kt.coil3:coil-network-okhttp:3.5.0")
}
```

---

## 📁 Estructura de proyecto

```
app/src/main/
├── java/com/empresa/app/
│   ├── App.kt                          # Application class (Koin init)
│   ├── MainActivity.kt
│   ├── core/
│   │   ├── di/                         # Módulos Koin globales
│   │   ├── network/                    # Retrofit, OkHttp
│   │   ├── database/                   # Room Database
│   │   └── datastore/                  # DataStore preferences
│   └── feature/
│       └── products/
│           ├── data/
│           │   ├── local/              # DAO, entidades Room
│           │   ├── remote/             # API service, DTOs
│           │   └── repository/         # Implementación del repo
│           ├── domain/
│           │   ├── model/              # Modelos de dominio
│           │   ├── repository/         # Interface del repo
│           │   └── usecase/            # Casos de uso
│           └── presentation/
│               ├── ProductsScreen.kt
│               ├── ProductsViewModel.kt
│               └── di/                 # Módulo Koin del feature
└── res/
    ├── values/
    │   ├── strings.xml
    │   └── themes.xml
    └── xml/
        └── backup_rules.xml
```

---

## 🚀 Punto de entrada

```kotlin
// App.kt
class App : Application() {
    override fun onCreate() {
        super.onCreate()
        startKoin {
            androidLogger(Level.DEBUG)
            androidContext(this@App)
            workManagerFactory()          // Para Workers con Koin
            modules(
                networkModule,
                databaseModule,
                datastoreModule,
                productsModule,
            )
        }
    }
}

// AndroidManifest.xml
// <application android:name=".App" ...>

// MainActivity.kt
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            MyAppTheme {
                AppNavGraph()
            }
        }
    }
}
```

