# Flutter — Base

---

## ⚙️ CLI — Comandos esenciales

```bash
flutter create <app_name>               # Crear proyecto nuevo
flutter create --org com.empresa <app>  # Con nombre de organización
flutter run                             # Correr en dispositivo/emulador
flutter run -d chrome                   # Correr en web
flutter run --release                   # Correr en modo release
flutter build apk                       # Build Android APK
flutter build apk --release             # APK optimizado
flutter build appbundle                 # Android App Bundle (Play Store)
flutter build ios                       # Build iOS
flutter build web                       # Build web
flutter test                            # Correr tests
flutter test --coverage                 # Con cobertura
flutter pub get                         # Instalar dependencias
flutter pub add <package>               # Agregar paquete
flutter pub remove <package>            # Quitar paquete
flutter pub outdated                    # Ver paquetes desactualizados
flutter pub upgrade                     # Actualizar paquetes
flutter doctor                          # Verificar entorno
flutter clean                           # Limpiar build cache
flutter analyze                         # Análisis estático del código
flutter gen-l10n                        # Generar archivos de localización
flutter upgrade                         # Actualizar Flutter
```



---

## 📁 Estructura de proyecto

```
my_app/
├── lib/
│   ├── main.dart               # Punto de entrada
│   ├── app.dart                # Widget raíz (MaterialApp)
│   ├── core/
│   │   ├── theme/              # ThemeData, colores, tipografía
│   │   ├── router/             # Rutas (GoRouter, etc.)
│   │   └── constants/          # Constantes globales
│   ├── features/
│   │   └── auth/
│   │       ├── data/           # Repositorios, fuentes de datos
│   │       ├── domain/         # Modelos, casos de uso
│   │       └── presentation/   # Widgets, pages, providers
│   └── shared/
│       ├── widgets/            # Widgets reutilizables
│       └── utils/              # Helpers y extensiones
├── test/                       # Unit y widget tests
├── assets/
│   ├── images/
│   └── fonts/
└── pubspec.yaml
```



---

## 🚀 Punto de entrada

```dart
// main.dart
void main() {
  WidgetsFlutterBinding.ensureInitialized(); // Necesario antes de código async
  runApp(const MyApp());
}

// app.dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Mi App',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const HomePage(),
      debugShowCheckedModeBanner: false,
    );
  }
}
```


