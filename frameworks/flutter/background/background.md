# Flutter — Tareas en background

---

## ⚙️ workmanager — tareas diferidas y periódicas

`workmanager` es un wrapper sobre WorkManager (Android) y BGTaskScheduler (iOS) para tareas garantizadas que sobreviven a que la app se cierre. **No es para trabajo en tiempo real** — el SO decide cuándo ejecutarlas según batería, red y otras políticas del dispositivo.

```dart
// pubspec.yaml
// workmanager: ^0.9.0+3

import 'package:workmanager/workmanager.dart';

// El callback dispatcher corre en un isolate aparte — debe ser una función top-level
// (o estática) y necesita el pragma para que no se elimine con tree-shaking en release
@pragma('vm:entry-point')
void callbackDispatcher() {
  Workmanager().executeTask((task, inputData) async {
    switch (task) {
      case 'syncTask':
        await syncPendingData();
        break;
      case 'cleanupTask':
        await clearOldCache();
        break;
    }
    return Future.value(true);   // true = éxito, false = reintentar según BackoffPolicy
  });
}

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Workmanager().initialize(callbackDispatcher, isInDebugMode: kDebugMode);
  runApp(const MyApp());
}

// Tarea única diferida
Workmanager().registerOneOffTask(
  'sync-once',
  'syncTask',
  initialDelay: const Duration(minutes: 5),
  constraints: Constraints(
    networkType: NetworkType.connected,
    requiresBatteryNotLow: true,
  ),
);

// Tarea periódica — Android exige un mínimo de 15 minutos entre ejecuciones
Workmanager().registerPeriodicTask(
  'sync-periodic',
  'syncTask',
  frequency: const Duration(hours: 1),
  constraints: Constraints(
    networkType: NetworkType.connected,
    requiresCharging: false,
  ),
  existingWorkPolicy: ExistingPeriodicWorkPolicy.keep,   // No duplicar si ya existe
);

// Cancelar
Workmanager().cancelByUniqueName('sync-periodic');
Workmanager().cancelAll();
```

```kotlin
// Android — AndroidManifest.xml no requiere configuración extra para workmanager,
// pero si el trabajo necesita seguir con la pantalla apagada, verificar que la app
// no esté en la lista de optimización de batería del fabricante (ver seguridad/permisos)
```

> ⚠️ **iOS es mucho más restrictivo**: `BGTaskScheduler` no garantiza que la tarea corra a la hora pedida — el sistema decide según uso de batería y patrones de uso de la app, y puede directamente no ejecutarla nunca si la app no se abre seguido. No depender de background tasks para algo crítico en iOS.

---

## 🔄 flutter_background_service — servicio en foreground continuo

Para trabajo que necesita correr de forma **continua** (no solo periódica) mientras la app está en background — ej: tracking de ubicación, reproducción de audio, un timer visible — usar `flutter_background_service`, que en Android corre como un foreground service con notificación persistente.

```dart
// pubspec.yaml
// flutter_background_service: ^5.1.0

import 'package:flutter_background_service/flutter_background_service.dart';

Future<void> initializeService() async {
  final service = FlutterBackgroundService();
  await service.configure(
    androidConfiguration: AndroidConfiguration(
      onStart: onStart,
      autoStart: false,           // Iniciar manualmente, no al abrir la app
      isForegroundMode: true,     // Requiere notificación persistente (ver notificaciones)
    ),
    iosConfiguration: IosConfiguration(
      autoStart: false,
      onForeground: onStart,
      onBackground: onIosBackground,
    ),
  );
}

@pragma('vm:entry-point')
void onStart(ServiceInstance service) async {
  DartPluginRegistrant.ensureInitialized();   // Necesario para usar plugins en el isolate del servicio

  Timer.periodic(const Duration(seconds: 30), (timer) async {
    final position = await getCurrentLocation();

    // Actualizar la notificación (solo Android)
    if (service is AndroidServiceInstance) {
      service.setForegroundNotificationInfo(
        title: 'Rastreo activo',
        content: 'Última posición: ${position.latitude}, ${position.longitude}',
      );
    }

    // Comunicar datos de vuelta a la UI
    service.invoke('update', {'lat': position.latitude, 'lng': position.longitude});
  });

  service.on('stop').listen((event) => service.stopSelf());
}

@pragma('vm:entry-point')
Future<bool> onIosBackground(ServiceInstance service) async => true;

// Iniciar/detener desde la UI
final service = FlutterBackgroundService();
await service.startService();
service.invoke('stop');

// Escuchar datos que manda el servicio
service.on('update').listen((event) {
  final lat = event?['lat'];
  final lng = event?['lng'];
});
```

> Requiere el permiso `FOREGROUND_SERVICE` (y el tipo específico, ej. `FOREGROUND_SERVICE_LOCATION`) en `AndroidManifest.xml` — ver la sección de permisos en `plataforma-nativa/plataforma-nativa.md`.
