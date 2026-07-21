# Flutter — Plataforma nativa

---

## 📱 Platform Channels

```dart
// Comunicación con código nativo (iOS/Android)

// MethodChannel — llamadas únicas
class PlatformService {
  static const _channel = MethodChannel('com.miapp/platform');

  // Llamar método nativo
  Future<String> getDeviceId() async {
    try {
      final id = await _channel.invokeMethod<String>('getDeviceId');
      return id ?? 'unknown';
    } on PlatformException catch (e) {
      throw Exception('Error nativo: ${e.message}');
    }
  }

  Future<bool> isRooted() async {
    return await _channel.invokeMethod<bool>('isRooted') ?? false;
  }

  Future<void> setStatusBarColor(Color color) async {
    await _channel.invokeMethod('setStatusBarColor', {
      'color': color.value,
      'isDark': color.computeLuminance() < 0.5,
    });
  }
}

// EventChannel — stream de eventos desde nativo
class AccelerometerService {
  static const _channel = EventChannel('com.miapp/accelerometer');

  Stream<Map<String, double>> get sensorStream {
    return _channel.receiveBroadcastStream().map((event) {
      final map = event as Map;
      return {
        'x': (map['x'] as num).toDouble(),
        'y': (map['y'] as num).toDouble(),
        'z': (map['z'] as num).toDouble(),
      };
    });
  }
}

// Android — MainActivity.kt
/*
class MainActivity : FlutterActivity() {
    private val channel = "com.miapp/platform"

    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)
        MethodChannel(flutterEngine.dartExecutor.binaryMessenger, channel)
            .setMethodCallHandler { call, result ->
                when (call.method) {
                    "getDeviceId" -> result.success(Settings.Secure.getString(
                        contentResolver, Settings.Secure.ANDROID_ID
                    ))
                    "isRooted" -> result.success(isRooted())
                    else -> result.notImplemented()
                }
            }
    }
}
*/

// iOS — AppDelegate.swift
/*
@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
    override func application(_ application: UIApplication,
        didFinishLaunchingWithOptions options: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        let controller = window?.rootViewController as! FlutterViewController
        let channel = FlutterMethodChannel(
            name: "com.miapp/platform",
            binaryMessenger: controller.binaryMessenger
        )
        channel.setMethodCallHandler { call, result in
            if call.method == "getDeviceId" {
                result(UIDevice.current.identifierForVendor?.uuidString ?? "unknown")
            } else {
                result(FlutterMethodNotImplemented)
            }
        }
        return super.application(application, didFinishLaunchingWithOptions: options)
    }
}
*/
```



---

## 🔔 Notificaciones locales

```yaml
dependencies:
  flutter_local_notifications: ^22.1.0
  timezone: ^0.11.1
```

```dart
class LocalNotificationService {
  static final _plugin = FlutterLocalNotificationsPlugin();

  static Future<void> initialize() async {
    const androidSettings = AndroidInitializationSettings('@mipmap/ic_launcher');
    const iosSettings = DarwinInitializationSettings(
      requestAlertPermission: true,
      requestBadgePermission: true,
      requestSoundPermission: true,
    );

    await _plugin.initialize(
      const InitializationSettings(android: androidSettings, iOS: iosSettings),
      onDidReceiveNotificationResponse: (response) {
        // Manejar tap en notificación
        final payload = response.payload;
        if (payload != null) navigatorKey.currentContext?.go(payload);
      },
    );

    // Crear canales en Android
    await _plugin
        .resolvePlatformSpecificImplementation<AndroidFlutterLocalNotificationsPlugin>()
        ?.createNotificationChannel(const AndroidNotificationChannel(
          'high_importance_channel',
          'Notificaciones importantes',
          importance: Importance.high,
          playSound: true,
        ));
  }

  // Notificación inmediata
  static Future<void> show({
    required int id,
    required String title,
    required String body,
    String? payload,
  }) async {
    await _plugin.show(
      id,
      title,
      body,
      NotificationDetails(
        android: AndroidNotificationDetails(
          'high_importance_channel',
          'Notificaciones importantes',
          importance: Importance.high,
          priority: Priority.high,
          icon: '@mipmap/ic_launcher',
          largeIcon: const DrawableResourceAndroidBitmap('@mipmap/ic_launcher'),
        ),
        iOS: const DarwinNotificationDetails(
          presentAlert: true,
          presentBadge: true,
          presentSound: true,
        ),
      ),
      payload: payload,
    );
  }

  // Notificación programada
  static Future<void> schedule({
    required int id,
    required String title,
    required String body,
    required DateTime scheduledDate,
    String? payload,
  }) async {
    await _plugin.zonedSchedule(
      id,
      title,
      body,
      tz.TZDateTime.from(scheduledDate, tz.local),
      NotificationDetails(
        android: AndroidNotificationDetails(
          'scheduled_channel', 'Recordatorios',
          importance: Importance.high,
        ),
        iOS: const DarwinNotificationDetails(),
      ),
      androidScheduleMode: AndroidScheduleMode.exactAllowWhileIdle,
      uiLocalNotificationDateInterpretation:
          UILocalNotificationDateInterpretation.absoluteTime,
      payload: payload,
    );
  }

  // Cancelar
  static Future<void> cancel(int id) => _plugin.cancel(id);
  static Future<void> cancelAll() => _plugin.cancelAll();
}
```



---

## 📍 Permisos, cámara y geolocalización

```yaml
dependencies:
  permission_handler: ^12.0.3
  image_picker: ^1.2.3
  geolocator: ^14.0.3
  geocoding: ^5.0.0
```

```dart
// Permisos con permission_handler
class PermissionService {
  // Verificar y solicitar
  static Future<bool> requestCamera() => _request(Permission.camera);
  static Future<bool> requestMicrophone() => _request(Permission.microphone);
  static Future<bool> requestLocation() => _request(Permission.location);
  static Future<bool> requestLocationAlways() =>
      _request(Permission.locationAlways);
  static Future<bool> requestPhotos() => _request(Permission.photos);      // READ_MEDIA_IMAGES (Android 13+)
  static Future<bool> requestVideos() => _request(Permission.videos);      // READ_MEDIA_VIDEO (Android 13+)
  static Future<bool> requestNotifications() =>
      _request(Permission.notification);

  static Future<bool> _request(Permission permission) async {
    final status = await permission.status;
    if (status.isGranted) return true;
    if (status.isPermanentlyDenied) {
      await openAppSettings();    // Redirigir a ajustes
      return false;
    }
    return (await permission.request()).isGranted;
  }

  // Verificar múltiples permisos
  static Future<Map<Permission, bool>> requestMultiple(
    List<Permission> permissions,
  ) async {
    final statuses = await permissions.request();
    return statuses.map((k, v) => MapEntry(k, v.isGranted));
  }
}

// Cámara y galería
class MediaPickerService {
  static final _picker = ImagePicker();

  // Seleccionar imagen de galería
  static Future<File?> pickImage({
    double? maxWidth = 1024,
    double? maxHeight = 1024,
    int? imageQuality = 85,
  }) async {
    if (!await PermissionService.requestPhotos()) return null;

    final picked = await _picker.pickImage(
      source: ImageSource.gallery,
      maxWidth: maxWidth,
      maxHeight: maxHeight,
      imageQuality: imageQuality,
    );
    return picked != null ? File(picked.path) : null;
  }

  // Capturar con cámara
  static Future<File?> capturePhoto({int? imageQuality = 85}) async {
    if (!await PermissionService.requestCamera()) return null;

    final picked = await _picker.pickImage(
      source: ImageSource.camera,
      imageQuality: imageQuality,
    );
    return picked != null ? File(picked.path) : null;
  }

  // Múltiples imágenes
  static Future<List<File>> pickMultipleImages() async {
    if (!await PermissionService.requestPhotos()) return [];

    final picked = await _picker.pickMultiImage(imageQuality: 85);
    return picked.map((x) => File(x.path)).toList();
  }

  // Video
  static Future<File?> pickVideo() async {
    final picked = await _picker.pickVideo(
      source: ImageSource.gallery,
      maxDuration: const Duration(minutes: 5),
    );
    return picked != null ? File(picked.path) : null;
  }
}

// Geolocalización
class LocationService {
  static Future<bool> isServiceEnabled() =>
      Geolocator.isLocationServiceEnabled();

  static Future<Position?> getCurrentLocation() async {
    if (!await PermissionService.requestLocation()) return null;
    if (!await isServiceEnabled()) return null;

    return Geolocator.getCurrentPosition(
      locationSettings: const LocationSettings(
        accuracy: LocationAccuracy.high,
        timeLimit: Duration(seconds: 10),
      ),
    );
  }

  static Stream<Position> getLocationStream() => Geolocator.getPositionStream(
    locationSettings: const LocationSettings(
      accuracy: LocationAccuracy.high,
      distanceFilter: 10,    // Actualizar cada 10 metros
    ),
  );

  static Future<double> getDistanceBetween({
    required double startLat,
    required double startLng,
    required double endLat,
    required double endLng,
  }) async => Geolocator.distanceBetween(startLat, startLng, endLat, endLng);

  // Geocoding — coordenadas ↔ dirección
  static Future<String> getAddressFromCoords(double lat, double lng) async {
    final placemarks = await placemarkFromCoordinates(lat, lng);
    final place = placemarks.first;
    return '${place.street}, ${place.locality}, ${place.country}';
  }

  static Future<Position?> getPositionFromAddress(String address) async {
    final locations = await locationFromAddress(address);
    if (locations.isEmpty) return null;
    return Position(
      latitude: locations.first.latitude,
      longitude: locations.first.longitude,
      timestamp: DateTime.now(),
      accuracy: 0, altitude: 0, heading: 0, speed: 0, speedAccuracy: 0,
      altitudeAccuracy: 0, headingAccuracy: 0,
    );
  }
}
```



---

## 📶 Connectivity

```yaml
dependencies:
  connectivity_plus: ^7.3.0
```

```dart
// ConnectivityCubit — expone si hay conexión como bool
class ConnectivityCubit extends Cubit<bool> {
  ConnectivityCubit() : super(true) {
    _subscription = Connectivity().onConnectivityChanged.listen((results) {
      emit(!results.contains(ConnectivityResult.none));
    });
  }

  late final StreamSubscription<List<ConnectivityResult>> _subscription;

  @override
  Future<void> close() {
    _subscription.cancel();
    return super.close();
  }
}

// Banner de sin conexión
class ConnectivityBanner extends StatelessWidget {
  const ConnectivityBanner({super.key, required this.child});
  final Widget child;

  @override
  Widget build(BuildContext context) {
    return BlocBuilder<ConnectivityCubit, bool>(
      builder: (context, isOnline) => Column(
        children: [
          AnimatedContainer(
            duration: const Duration(milliseconds: 300),
            height: isOnline ? 0 : 40,
            color: Colors.red.shade700,
            child: const Center(
              child: Text(
                "Sin conexión a internet",
                style: TextStyle(color: Colors.white, fontSize: 13),
              ),
            ),
          ),
          Expanded(child: child),
        ],
      ),
    );
  }
}

// Verificar conectividad antes de una operación
Future<Result<T>> withConnectivity<T>(
  ConnectivityCubit connectivityCubit,
  Future<Result<T>> Function() operation,
) async {
  if (!connectivityCubit.state) {
    return Failure("Sin conexión a internet");
  }
  return operation();
}
```


