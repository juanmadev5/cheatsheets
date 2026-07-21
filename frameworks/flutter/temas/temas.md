# Flutter — Temas y estilos

---

## 🎨 Temas y estilos

```dart
// app_theme.dart
class AppTheme {
  static ThemeData get light => ThemeData(
    useMaterial3: true,
    colorScheme: ColorScheme.fromSeed(
      seedColor: const Color(0xFF2563EB),
      brightness: Brightness.light,
    ),
    textTheme: const TextTheme(
      displayLarge: TextStyle(fontSize: 32, fontWeight: FontWeight.bold),
      bodyLarge: TextStyle(fontSize: 16, height: 1.5),
    ),
    appBarTheme: const AppBarTheme(centerTitle: true, elevation: 0),
    inputDecorationTheme: InputDecorationTheme(
      border: OutlineInputBorder(borderRadius: BorderRadius.circular(8)),
      filled: true,
    ),
    elevatedButtonTheme: ElevatedButtonThemeData(
      style: ElevatedButton.styleFrom(
        minimumSize: const Size.fromHeight(48),
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(8)),
      ),
    ),
  );

  static ThemeData get dark => ThemeData(
    useMaterial3: true,
    colorScheme: ColorScheme.fromSeed(
      seedColor: const Color(0xFF2563EB),
      brightness: Brightness.dark,
    ),
  );
}

// Usar tema en toda la app
MaterialApp(
  theme: AppTheme.light,
  darkTheme: AppTheme.dark,
  themeMode: ThemeMode.system,   // .light | .dark | .system
)

// Acceder al tema en widgets
final theme = Theme.of(context);
final colorScheme = theme.colorScheme;
Text('Hola', style: theme.textTheme.headlineMedium)
Container(color: colorScheme.primaryContainer)
```



---

## 🗨️ Diálogos, Sheets y Snackbars

```dart
// AlertDialog
showDialog(
  context: context,
  builder: (context) => AlertDialog(
    title: const Text('Confirmar'),
    content: const Text('¿Estás seguro?'),
    actions: [
      TextButton(onPressed: () => Navigator.pop(context, false), child: const Text('Cancelar')),
      ElevatedButton(onPressed: () => Navigator.pop(context, true), child: const Text('Confirmar')),
    ],
  ),
);

// BottomSheet modal
showModalBottomSheet(
  context: context,
  isScrollControlled: true,    // Para sheets más altos
  shape: const RoundedRectangleBorder(
    borderRadius: BorderRadius.vertical(top: Radius.circular(16)),
  ),
  builder: (context) => DraggableScrollableSheet(
    expand: false,
    builder: (_, controller) => ListView(controller: controller, children: [/* ... */]),
  ),
);

// SnackBar
ScaffoldMessenger.of(context).showSnackBar(
  SnackBar(
    content: const Text('Operación exitosa'),
    action: SnackBarAction(label: 'Deshacer', onPressed: () {}),
    behavior: SnackBarBehavior.floating,
    duration: const Duration(seconds: 3),
  ),
);
```


