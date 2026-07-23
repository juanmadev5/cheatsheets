# Kotlin — Build — build.gradle.kts

---

## 🛠️ build.gradle.kts

```kotlin
plugins {
    kotlin("jvm") version "2.3.20"
    kotlin("plugin.serialization") version "2.3.20"
    id("com.google.devtools.ksp") version "2.3.10"  // KSP — versionado propio desde 2.x, compatible con Kotlin 2.2+
    application
}

group = "com.miempresa"
version = "1.0.0"

kotlin {
    jvmToolchain(25)   // Java 25 (LTS) — más moderno que compileKotlin { kotlinOptions { jvmTarget } }
}

repositories {
    mavenCentral()
}

dependencies {
    // Coroutines
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.11.0")

    // Kotlin Serialization (alternativa a Gson/Jackson)
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.11.0")

    // Ktor — HTTP client/server puro Kotlin
    implementation("io.ktor:ktor-client-core:3.5.0")
    implementation("io.ktor:ktor-client-cio:3.5.0")
    implementation("io.ktor:ktor-client-content-negotiation:3.5.0")
    implementation("io.ktor:ktor-serialization-kotlinx-json:3.5.0")
    implementation("io.ktor:ktor-client-logging:3.5.0")

    // Arrow — programación funcional en Kotlin (2.x requiere Kotlin 2.2+)
    implementation("io.arrow-kt:arrow-core:2.2.3")
    implementation("io.arrow-kt:arrow-fx-coroutines:2.2.3")

    // Logging
    implementation("io.github.oshai:kotlin-logging-jvm:8.0.4")
    implementation("ch.qos.logback:logback-classic:1.5.38")

    // Testing
    testImplementation(kotlin("test"))
    testImplementation("org.jetbrains.kotlinx:kotlinx-coroutines-test:1.11.0")
    testImplementation("io.mockk:mockk:1.14.7")
    // Kotest 6.x requiere JDK 11+ y Kotlin 2.2+; @AutoScan fue removido —
    // registrar extensions explícitamente vía io.kotest.provided.ProjectConfig
    testImplementation("io.kotest:kotest-runner-junit5:6.2.3")
    testImplementation("io.kotest:kotest-assertions-core:6.2.3")
    testImplementation("io.kotest:kotest-property:6.2.3")   // Property-based testing
}

tasks.test {
    useJUnitPlatform()
}

application {
    mainClass.set("com.miempresa.MainKt")
}

// Tarea de fat JAR
tasks.jar {
    manifest { attributes["Main-Class"] = "com.miempresa.MainKt" }
    from(configurations.runtimeClasspath.get().map { if (it.isDirectory) it else zipTree(it) })
    duplicatesStrategy = DuplicatesStrategy.EXCLUDE
}
```
