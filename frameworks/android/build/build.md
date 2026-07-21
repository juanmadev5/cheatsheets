# Android Jetpack Compose — Build y distribución

---

## 🏗️ Build Variants y Flavors

```kotlin
// build.gradle.kts (app) — configuración completa

android {
    // Signing configs
    signingConfigs {
        create("release") {
            storeFile = file(System.getenv("KEYSTORE_PATH") ?: "keystore.jks")
            storePassword = System.getenv("KEYSTORE_PASS") ?: ""
            keyAlias = System.getenv("KEY_ALIAS") ?: ""
            keyPassword = System.getenv("KEY_PASS") ?: ""
        }
    }

    buildTypes {
        debug {
            applicationIdSuffix = ".debug"
            versionNameSuffix = "-debug"
            isDebuggable = true
            isMinifyEnabled = false
            buildConfigField("String", "API_BASE_URL", "\"https://dev.api.miapp.com/v1/\"")
            buildConfigField("Boolean", "ENABLE_LOGGING", "true")
            resValue("string", "app_name", "Mi App (Debug)")
        }
        release {
            isMinifyEnabled = true
            isShrinkResources = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
            signingConfig = signingConfigs.getByName("release")
            buildConfigField("String", "API_BASE_URL", "\"https://api.miapp.com/v1/\"")
            buildConfigField("Boolean", "ENABLE_LOGGING", "false")
        }
        create("staging") {
            initWith(getByName("release"))          // Hereda de release
            applicationIdSuffix = ".staging"
            versionNameSuffix = "-staging"
            isDebuggable = true
            buildConfigField("String", "API_BASE_URL", "\"https://staging.api.miapp.com/v1/\"")
            buildConfigField("Boolean", "ENABLE_LOGGING", "true")
        }
    }

    // Product Flavors — variantes del producto
    flavorDimensions += listOf("tier", "store")

    productFlavors {
        create("free") {
            dimension = "tier"
            applicationIdSuffix = ".free"
            buildConfigField("Boolean", "IS_PRO", "false")
            buildConfigField("Int", "MAX_ITEMS", "10")
        }
        create("pro") {
            dimension = "tier"
            applicationIdSuffix = ".pro"
            buildConfigField("Boolean", "IS_PRO", "true")
            buildConfigField("Int", "MAX_ITEMS", "Int.MAX_VALUE")
        }
        create("playStore") {
            dimension = "store"
        }
        create("huaweiStore") {
            dimension = "store"
            buildConfigField("String", "STORE", "\"huawei\"")
        }
    }

    // Habilitar BuildConfig
    buildFeatures {
        buildConfig = true
        compose = true
    }
}
```

```kotlin
// Usar BuildConfig en código
val apiUrl = BuildConfig.API_BASE_URL
val isPro = BuildConfig.IS_PRO
val enableLogging = BuildConfig.ENABLE_LOGGING

// Fuentes de código por flavor: src/free/kotlin/, src/pro/kotlin/
// Fuentes de recursos por flavor: src/free/res/, src/pro/res/
// Fuente compartida sigue en: src/main/kotlin/, src/main/res/
```

---

## 🛡️ ProGuard / R8 — reglas esenciales

```pro
# proguard-rules.pro

# Kotlin
-keep class kotlin.** { *; }
-keepclassmembers class **$WhenMappings { *; }
-keepclassmembers class kotlin.Metadata { *; }

# Kotlinx Serialization
-keepattributes *Annotation*, InnerClasses
-dontnote kotlinx.serialization.AnnotationsKt
-keepclassmembers class kotlinx.serialization.json.** { *** Companion; }
-keepclasseswithmembers class **$$serializer { *; }
@kotlinx.serialization.Serializable class * { *; }

# Retrofit
-keepattributes Signature, Exceptions
-keepclassmembers,allowshrinking,allowobfuscation interface * {
    @retrofit2.http.* <methods>;
}
-dontwarn org.codehaus.mojo.animal_sniffer.IgnoreJRERequirement
-dontwarn javax.annotation.**

# OkHttp
-dontwarn okhttp3.**
-dontwarn okio.**
-keepnames class okhttp3.internal.publicsuffix.PublicSuffixDatabase

# Room
-keep class * extends androidx.room.RoomDatabase
-keep @androidx.room.Entity class *
-dontwarn androidx.room.paging.**

# Koin
-keepclassmembers class * { @org.koin.core.annotation.* <fields>; }

# Firebase
-keep class com.google.firebase.** { *; }
-dontwarn com.google.firebase.**

# Coil — no requiere reglas manuales, trae sus propias consumer-rules en el AAR

# Parcelable
-keepclassmembers class * implements android.os.Parcelable {
    static ** CREATOR;
}

# Enums
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

# Crashlytics
-keepattributes SourceFile, LineNumberTable
-keep public class * extends java.lang.Exception
-keep class com.google.firebase.crashlytics.** { *; }

# Modelos de datos (si usás reflexión) — ajustar al paquete propio
-keep class com.tuempresa.tuapp.data.model.** { *; }

# Mantener stack traces legibles (para Crashlytics)
-keepattributes SourceFile, LineNumberTable
-renamesourcefileattribute SourceFile
```

