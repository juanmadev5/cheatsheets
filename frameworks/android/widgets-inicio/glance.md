# Android Jetpack Compose — Widgets de pantalla de inicio

---

## 📱 App Widget con Glance

```kotlin
// build.gradle.kts
// implementation("androidx.glance:glance-appwidget:1.1.1")
// implementation("androidx.glance:glance-material3:1.1.1")

// Widget básico con Glance (Compose-like API)
class StatsWidget : GlanceAppWidget() {
    override suspend fun provideGlance(context: Context, id: GlanceId) {
        val stats = StatsRepository(context).getStats()   // cargar datos

        provideContent {
            GlanceTheme {
                StatsWidgetContent(stats = stats)
            }
        }
    }
}

@Composable
fun StatsWidgetContent(stats: Stats) {
    Column(
        modifier = GlanceModifier
            .fillMaxSize()
            .background(GlanceTheme.colors.surface)
            .padding(16.dp),
        verticalAlignment = Alignment.Vertical.CenterVertically,
    ) {
        Text(
            text = stats.title,
            style = TextStyle(
                fontWeight = FontWeight.Bold,
                fontSize = 16.sp,
                color = GlanceTheme.colors.onSurface,
            ),
        )
        Spacer(modifier = GlanceModifier.height(8.dp))
        Text(
            text = stats.value,
            style = TextStyle(fontSize = 32.sp, color = GlanceTheme.colors.primary),
        )
        // Botón que actualiza el widget
        Button(
            text = "Actualizar",
            onClick = actionRunCallback<RefreshWidgetAction>(),
        )
        // Botón que abre la app
        Button(
            text = "Abrir app",
            onClick = actionStartActivity<MainActivity>(),
        )
    }
}

// Acción del widget
class RefreshWidgetAction : ActionCallback {
    override suspend fun onAction(context: Context, glanceId: GlanceId, parameters: ActionParameters) {
        StatsWidget().update(context, glanceId)
    }
}

// Receiver — registro del widget
class StatsWidgetReceiver : GlanceAppWidgetReceiver() {
    override val glanceAppWidget: GlanceAppWidget = StatsWidget()
}
```

```xml
<!-- AndroidManifest.xml -->
<receiver
    android:name=".widget.StatsWidgetReceiver"
    android:exported="true">
    <intent-filter>
        <action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
    </intent-filter>
    <meta-data
        android:name="android.appwidget.provider"
        android:resource="@xml/stats_widget_info" />
</receiver>

<!-- res/xml/stats_widget_info.xml -->
```
```xml
<appwidget-provider
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:minWidth="180dp"
    android:minHeight="110dp"
    android:targetCellWidth="2"
    android:targetCellHeight="2"
    android:updatePeriodMillis="1800000"
    android:resizeMode="horizontal|vertical"
    android:widgetCategory="home_screen"
    android:initialLayout="@layout/glance_default_loading_layout" />
```

