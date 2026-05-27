# Flutter — Cheatsheet

---

## 📑 Índice

### Base
- [CLI — Comandos esenciales](#️-cli--comandos-esenciales)
- [Estructura de proyecto](#-estructura-de-proyecto)
- [Punto de entrada](#-punto-de-entrada)

### Dart moderno — Dart 3.x
- [Null safety](#null-safety)
- [Records y destructuring](#records-y-destructuring-dart-3)
- [Pattern matching y sealed classes](#pattern-matching-y-sealed-classes-dart-3)
- [Extensions, Mixins y Generics](#extensions-mixins-y-generics)
- [Modelos con freezed + json_serializable](#modelos-con-freezed--json_serializable)

### Widgets y UI
- [Widgets fundamentales — StatelessWidget vs StatefulWidget](#-widgets-fundamentales)
- [Layout — Column, Row, Stack, Expanded, Container](#-layout--widgets-de-estructura)
- [Widgets de listas — ListView, GridView](#-widgets-de-listas)
- [Widgets visuales — Text, Imagen, Botones, Forms](#-widgets-visuale-comunes)
- [Scaffold y navegación básica](#-scaffold-y-navegación-básica)
- [Ciclo de vida de widgets](#-ciclo-de-vida-de-widgets)

### UI avanzada
- [CustomPainter](#custompainter)
- [Slivers — scroll personalizado](#slivers--scroll-personalizado)
- [LayoutBuilder y MediaQuery — responsive](#layoutbuilder-y-mediaquery--diseño-responsive)
- [InheritedWidget](#inheritedwidget--compartir-datos-por-el-árbol)

### Navegación
- [GoRouter — básico](#️-gorouter-navegación-declarativa)
- [GoRouter avanzado — StatefulShellRoute, guards, deep links](#️-gorouter-avanzado)

### Estado
- [Provider — ChangeNotifier, FutureProvider, StreamProvider](#-manejo-de-estado-con-provider)
- [Riverpod — básico](#-riverpod-alternativa-moderna-a-provider)
- [Riverpod — completo con code generation](#-riverpod--completo-con-code-generation)
- [Riverpod — AsyncNotifier, Notifier, StreamNotifier](#asyncnotifier-y-notifier)
- [Riverpod — consumir providers en widgets](#consumir-providers-en-widgets)
- [BLoC — Cubit y Bloc](#-bloc-pattern)
- [BLoC — en la UI (Builder, Listener, Consumer)](#bloc-en-la-ui)

### Red y APIs
- [HTTP y APIs — básico](#-http-y-apis)
- [Dio — completo con interceptores y refresh token](#-dio--http-completo)
- [FutureBuilder y StreamBuilder](#-manejo-de-async--futurebuilder-y-streambuilder)

### Firebase
- [Setup y Crashlytics](#-firebase-completo)
- [Firebase Auth — email, Google, Apple](#firebase-auth)
- [Firestore — CRUD, streams, transacciones, paginación](#firestore-crud--streams)
- [Firebase Storage](#firebase-storage)
- [Firebase Cloud Messaging — push notifications](#firebase-cloud-messaging-push-notifications)

### Persistencia
- [Persistencia local — SharedPreferences, Hive](#-persistencia-local)
- [Isar — base de datos local moderna](#️-isar--base-de-datos-local-moderna)

### Arquitectura
- [Clean Architecture — estructura, repositorios, use cases, get_it](#️-clean-architecture)

### Plataforma nativa
- [Platform Channels — MethodChannel, EventChannel](#-platform-channels)
- [Permisos — permission_handler](#-permisos-cámara-y-geolocalización)
- [Cámara y galería — image_picker](#-permisos-cámara-y-geolocalización)
- [Geolocalización — geolocator, geocoding](#-permisos-cámara-y-geolocalización)
- [Notificaciones locales — flutter_local_notifications](#-notificaciones-locales)
- [Connectivity — monitoreo de red](#-connectivity)

### Animaciones
- [Animaciones básicas — AnimatedContainer, Hero, TweenAnimationBuilder](#-animaciones)
- [flutter_animate — animaciones declarativas](#flutter_animate)
- [AnimationController manual — Tween, AnimatedBuilder](#animationcontroller-manual)
- [Transiciones de ruta personalizadas](#animationcontroller-manual)

### Temas y estilos
- [Temas y estilos — ThemeData, Material 3, dark mode](#-temas-y-estilos)
- [Diálogos, Sheets y Snackbars](#️-diálogos-sheets-y-snackbars)

### Concurrencia
- [Isolates y compute](#-isolates-y-compute)

### Seguridad
- [flutter_secure_storage, biometría, certificate pinning, ofuscación](#-seguridad)

### Internacionalización
- [i18n — ARB files, plurales, fechas, monedas, idioma en runtime](#-internacionalización-i18n)

### Testing
- [Testing — básico](#-testing)
- [Testing completo — unit, BLoC, widget, golden tests](#-testing-completo)

### Performance
- [Performance — const, RepaintBoundary, KeepAlive, select](#-performance)

### Build y distribución
- [Flavors y build — dart-define-from-file, ofuscación, split APK](#️-flavors-y-build)

### Referencia
- [Paquetes esenciales](#-paquetes-esenciales)
- [Buenas prácticas](#-buenas-prácticas)

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

---

## 🧱 Widgets fundamentales

### StatelessWidget vs StatefulWidget

```dart
// StatelessWidget — sin estado interno
class MyWidget extends StatelessWidget {
  final String title;
  const MyWidget({super.key, required this.title});

  @override
  Widget build(BuildContext context) {
    return Text(title);
  }
}

// StatefulWidget — con estado interno mutable
class Counter extends StatefulWidget {
  const Counter({super.key});

  @override
  State<Counter> createState() => _CounterState();
}

class _CounterState extends State<Counter> {
  int _count = 0;

  @override
  Widget build(BuildContext context) {
    return TextButton(
      onPressed: () => setState(() => _count++),
      child: Text('Count: $_count'),
    );
  }
}
```

---

## 📐 Layout — Widgets de estructura

### Column y Row

```dart
Column(
  mainAxisAlignment: MainAxisAlignment.center,    // Eje principal (vertical)
  crossAxisAlignment: CrossAxisAlignment.start,   // Eje cruzado (horizontal)
  mainAxisSize: MainAxisSize.min,                 // Tamaño mínimo necesario
  children: [
    Text('Primero'),
    const SizedBox(height: 8),
    Text('Segundo'),
  ],
)

Row(
  mainAxisAlignment: MainAxisAlignment.spaceBetween,
  children: [
    Icon(Icons.home),
    Text('Título'),
    Icon(Icons.settings),
  ],
)
```

### Stack

```dart
Stack(
  alignment: Alignment.center,
  children: [
    Image.network('https://...'),
    Positioned(
      bottom: 16,
      right: 16,
      child: FloatingActionButton(onPressed: () {}, child: const Icon(Icons.add)),
    ),
  ],
)
```

### Expanded y Flexible

```dart
Row(
  children: [
    Expanded(child: Container(color: Colors.red)),     // Ocupa todo el espacio disponible
    Expanded(flex: 2, child: Container(color: Colors.blue)), // El doble de espacio
    Flexible(child: Text('Solo lo necesario')),        // Solo el espacio que necesita
  ],
)
```

### Container

```dart
Container(
  width: 200,
  height: 100,
  margin: const EdgeInsets.all(16),
  padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 8),
  decoration: BoxDecoration(
    color: Colors.blue.shade100,
    borderRadius: BorderRadius.circular(12),
    border: Border.all(color: Colors.blue, width: 1.5),
    boxShadow: [
      BoxShadow(color: Colors.black26, blurRadius: 8, offset: Offset(0, 4)),
    ],
  ),
  child: const Text('Hola'),
)
```

### Otros widgets de layout

| Widget | Descripción |
|---|---|
| `SizedBox(width, height)` | Espacio fijo o contenedor de tamaño exacto |
| `Padding(padding:, child:)` | Agregar padding |
| `Center(child:)` | Centrar su hijo |
| `Align(alignment:, child:)` | Alinear hijo en posición específica |
| `AspectRatio(aspectRatio:)` | Mantener proporción |
| `FractionallySizedBox` | Tamaño como fracción del padre |
| `ConstrainedBox` | Aplicar constraints mínimos/máximos |
| `Wrap` | Como Row/Column pero con wrap |
| `Spacer()` | Espacio flexible entre widgets |
| `IntrinsicHeight` / `IntrinsicWidth` | Igualar alturas/anchos de hijos |

---

## 📋 Widgets de listas

```dart
// Lista simple
ListView(
  padding: const EdgeInsets.all(16),
  children: [Text('Item 1'), Text('Item 2')],
)

// Lista eficiente (lazy) — SIEMPRE usar para listas largas
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) => ListTile(
    leading: const Icon(Icons.person),
    title: Text(items[index].name),
    subtitle: Text(items[index].email),
    trailing: const Icon(Icons.chevron_right),
    onTap: () {},
  ),
)

// Lista separada
ListView.separated(
  itemCount: items.length,
  separatorBuilder: (_, __) => const Divider(),
  itemBuilder: (context, index) => Text(items[index]),
)

// Grid
GridView.builder(
  gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
    crossAxisCount: 2,
    childAspectRatio: 1.5,
    crossAxisSpacing: 8,
    mainAxisSpacing: 8,
  ),
  itemCount: items.length,
  itemBuilder: (context, index) => Card(child: Text(items[index])),
)
```

---

## 🎨 Widgets visuales comunes

### Text

```dart
Text(
  'Hola mundo',
  style: TextStyle(
    fontSize: 24,
    fontWeight: FontWeight.bold,
    color: Colors.blue,
    letterSpacing: 1.2,
    height: 1.5,            // Line height
    decoration: TextDecoration.underline,
  ),
  textAlign: TextAlign.center,
  maxLines: 2,
  overflow: TextOverflow.ellipsis,
)

// Rich text con múltiples estilos
RichText(
  text: TextSpan(
    text: 'Hola ',
    style: DefaultTextStyle.of(context).style,
    children: [
      TextSpan(text: 'mundo', style: TextStyle(fontWeight: FontWeight.bold)),
      TextSpan(text: '!', style: TextStyle(color: Colors.red)),
    ],
  ),
)
```

### Imagen

```dart
Image.asset('assets/images/logo.png', width: 100, fit: BoxFit.contain)
Image.network('https://...', fit: BoxFit.cover, errorBuilder: (ctx, err, st) => Icon(Icons.error))

// Imagen circular
CircleAvatar(
  radius: 30,
  backgroundImage: NetworkImage('https://...'),
  child: null,   // Si se usa backgroundImage, child queda encima
)
```

### Botones

```dart
ElevatedButton(onPressed: () {}, child: const Text('Elevado'))
OutlinedButton(onPressed: () {}, child: const Text('Outlined'))
TextButton(onPressed: () {}, child: const Text('Text'))
IconButton(onPressed: () {}, icon: const Icon(Icons.favorite))
FilledButton(onPressed: () {}, child: const Text('Filled'))  // Material 3

// Con ícono
ElevatedButton.icon(
  onPressed: () {},
  icon: const Icon(Icons.send),
  label: const Text('Enviar'),
)
```

### Input / Forms

```dart
// Campo de texto
TextField(
  controller: _controller,
  decoration: const InputDecoration(
    labelText: 'Email',
    hintText: 'tu@email.com',
    prefixIcon: Icon(Icons.email),
    border: OutlineInputBorder(),
    errorText: 'Campo requerido',   // null = sin error
  ),
  keyboardType: TextInputType.emailAddress,
  obscureText: false,
  maxLines: 1,
  onChanged: (value) {},
  onSubmitted: (value) {},
)

// Form con validación
final _formKey = GlobalKey<FormState>();

Form(
  key: _formKey,
  child: Column(
    children: [
      TextFormField(
        validator: (value) {
          if (value == null || value.isEmpty) return 'Campo requerido';
          return null;  // null = válido
        },
        decoration: const InputDecoration(labelText: 'Nombre'),
      ),
      ElevatedButton(
        onPressed: () {
          if (_formKey.currentState!.validate()) {
            // Formulario válido
          }
        },
        child: const Text('Enviar'),
      ),
    ],
  ),
)
```

---

## 🧭 Scaffold y navegación básica

```dart
Scaffold(
  appBar: AppBar(
    title: const Text('Mi Página'),
    actions: [IconButton(icon: const Icon(Icons.search), onPressed: () {})],
    leading: const BackButton(),
  ),
  body: const Center(child: Text('Contenido')),
  floatingActionButton: FloatingActionButton(
    onPressed: () {},
    child: const Icon(Icons.add),
  ),
  bottomNavigationBar: BottomNavigationBar(
    currentIndex: _index,
    onTap: (i) => setState(() => _index = i),
    items: const [
      BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Inicio'),
      BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Perfil'),
    ],
  ),
  drawer: Drawer(child: ListView(/* ... */)),
)
```

### Navigator (navegación imperativa)

```dart
// Ir a otra pantalla
Navigator.push(context, MaterialPageRoute(builder: (_) => const DetailPage()));

// Ir y reemplazar pantalla actual
Navigator.pushReplacement(context, MaterialPageRoute(builder: (_) => const HomePage()));

// Ir y eliminar todas las rutas anteriores
Navigator.pushAndRemoveUntil(
  context,
  MaterialPageRoute(builder: (_) => const HomePage()),
  (route) => false,
);

// Volver
Navigator.pop(context);

// Volver con resultado
Navigator.pop(context, 'resultado');

// Esperar resultado
final result = await Navigator.push(context, MaterialPageRoute(builder: (_) => const Picker()));
```

---

## 🗺️ GoRouter (navegación declarativa)

```yaml
# pubspec.yaml
dependencies:
  go_router: ^14.0.0
```

```dart
// router.dart
final router = GoRouter(
  initialLocation: '/',
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomePage(),
    ),
    GoRoute(
      path: '/detail/:id',
      builder: (context, state) {
        final id = state.pathParameters['id']!;
        return DetailPage(id: id);
      },
    ),
    ShellRoute(
      builder: (context, state, child) => MainShell(child: child),
      routes: [
        GoRoute(path: '/home', builder: (_, __) => const HomeTab()),
        GoRoute(path: '/profile', builder: (_, __) => const ProfileTab()),
      ],
    ),
  ],
  redirect: (context, state) {
    final isLoggedIn = AuthService.isLoggedIn;
    if (!isLoggedIn && state.matchedLocation != '/login') return '/login';
    return null;
  },
);

// Uso en MaterialApp
MaterialApp.router(routerConfig: router)

// Navegar
context.go('/detail/123');           // Reemplaza la pila
context.push('/detail/123');         // Apila sobre la actual
context.pop();                        // Volver
context.goNamed('detail', pathParameters: {'id': '123'});

// Leer parámetros
GoRoute(
  path: '/search',
  builder: (context, state) {
    final query = state.uri.queryParameters['q'] ?? '';
    return SearchPage(query: query);
  },
)
```

---

## 🔄 Manejo de estado con Provider

```yaml
# pubspec.yaml
dependencies:
  provider: ^6.1.0
```

### ChangeNotifier básico

```dart
// counter_provider.dart
class CounterProvider extends ChangeNotifier {
  int _count = 0;
  int get count => _count;

  void increment() {
    _count++;
    notifyListeners();   // Notifica a los widgets que escuchan
  }

  void decrement() {
    _count--;
    notifyListeners();
  }

  void reset() {
    _count = 0;
    notifyListeners();
  }
}

// main.dart — registrar el provider
void main() {
  runApp(
    ChangeNotifierProvider(
      create: (_) => CounterProvider(),
      child: const MyApp(),
    ),
  );
}

// Consumir el provider
class CounterPage extends StatelessWidget {
  const CounterPage({super.key});

  @override
  Widget build(BuildContext context) {
    // Escucha cambios y reconstruye
    final count = context.watch<CounterProvider>().count;

    return Column(
      children: [
        Text('$count'),
        // Solo lee sin escuchar (no reconstruye)
        ElevatedButton(
          onPressed: () => context.read<CounterProvider>().increment(),
          child: const Text('Incrementar'),
        ),
        // Escucha solo una propiedad específica
        Consumer<CounterProvider>(
          builder: (context, provider, child) => Text('${provider.count}'),
        ),
      ],
    );
  }
}
```

### Múltiples providers

```dart
void main() {
  runApp(
    MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (_) => AuthProvider()),
        ChangeNotifierProvider(create: (_) => CartProvider()),
        ChangeNotifierProxyProvider<AuthProvider, UserProvider>(
          create: (_) => UserProvider(),
          update: (_, auth, user) => user!..updateAuth(auth),
        ),
      ],
      child: const MyApp(),
    ),
  );
}
```

### Provider con async (FutureProvider / StreamProvider)

```dart
FutureProvider<List<User>>(
  create: (_) => UserRepository().fetchAll(),
  initialData: const [],
  child: const UserList(),
)

StreamProvider<int>(
  create: (_) => Stream.periodic(Duration(seconds: 1), (i) => i),
  initialData: 0,
  child: const TimerWidget(),
)
```

---

## ⚡ Riverpod (alternativa moderna a Provider)

```yaml
dependencies:
  flutter_riverpod: ^2.5.0
  riverpod_annotation: ^2.3.0
dev_dependencies:
  build_runner: ^2.4.0
  riverpod_generator: ^2.3.0
```

```dart
// main.dart
void main() {
  runApp(const ProviderScope(child: MyApp()));
}

// providers.dart — sintaxis con anotaciones (recomendada)
part 'providers.g.dart';

@riverpod
int counter(CounterRef ref) => 0;   // Provider simple

@riverpod
class CounterNotifier extends _$CounterNotifier {
  @override
  int build() => 0;    // Estado inicial

  void increment() => state++;
  void decrement() => state--;
}

@riverpod
Future<List<User>> users(UsersRef ref) async {
  return UserRepository().fetchAll();
}

// Consumir en widget — extender de ConsumerWidget
class MyPage extends ConsumerWidget {
  const MyPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterNotifierProvider);
    final usersAsync = ref.watch(usersProvider);

    return Column(
      children: [
        Text('$count'),
        ElevatedButton(
          onPressed: () => ref.read(counterNotifierProvider.notifier).increment(),
          child: const Text('+'),
        ),
        usersAsync.when(
          data: (users) => ListView.builder(
            itemCount: users.length,
            itemBuilder: (_, i) => Text(users[i].name),
          ),
          loading: () => const CircularProgressIndicator(),
          error: (err, _) => Text('Error: $err'),
        ),
      ],
    );
  }
}
```

---

## 🌊 Manejo de async — FutureBuilder y StreamBuilder

```dart
FutureBuilder<List<Post>>(
  future: ApiService().fetchPosts(),
  builder: (context, snapshot) {
    if (snapshot.connectionState == ConnectionState.waiting) {
      return const CircularProgressIndicator();
    }
    if (snapshot.hasError) return Text('Error: ${snapshot.error}');
    if (!snapshot.hasData || snapshot.data!.isEmpty) return const Text('Sin datos');

    final posts = snapshot.data!;
    return ListView.builder(
      itemCount: posts.length,
      itemBuilder: (_, i) => ListTile(title: Text(posts[i].title)),
    );
  },
)

StreamBuilder<int>(
  stream: timerStream,
  initialData: 0,
  builder: (context, snapshot) {
    return Text('${snapshot.data}');
  },
)
```

---

## 🎭 Ciclo de vida de widgets

```dart
class MyPage extends StatefulWidget {
  const MyPage({super.key});

  @override
  State<MyPage> createState() => _MyPageState();
}

class _MyPageState extends State<MyPage> {
  late TextEditingController _controller;
  late ScrollController _scrollController;

  @override
  void initState() {
    super.initState();                    // SIEMPRE llamar primero
    _controller = TextEditingController();
    _scrollController = ScrollController();
    _loadData();                          // Inicializar datos async
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();        // Llamado cuando dependencias cambian
    // Seguro acceder a context.watch() aquí
  }

  @override
  void didUpdateWidget(MyPage oldWidget) {
    super.didUpdateWidget(oldWidget);     // Widget padre reconstruyó con nuevos props
  }

  @override
  void dispose() {
    _controller.dispose();               // SIEMPRE liberar controllers
    _scrollController.dispose();
    super.dispose();                      // SIEMPRE llamar al final
  }

  Future<void> _loadData() async { /* ... */ }

  @override
  Widget build(BuildContext context) => Container();
}
```

---

## 🌐 HTTP y APIs

```yaml
dependencies:
  http: ^1.2.0
  dio: ^5.4.0          # Más potente que http
```

```dart
// Con http
import 'package:http/http.dart' as http;
import 'dart:convert';

Future<List<User>> fetchUsers() async {
  final response = await http.get(
    Uri.parse('https://api.example.com/users'),
    headers: {'Authorization': 'Bearer $token'},
  );

  if (response.statusCode == 200) {
    final List data = jsonDecode(response.body);
    return data.map((json) => User.fromJson(json)).toList();
  } else {
    throw Exception('Error ${response.statusCode}');
  }
}

// Modelo con fromJson / toJson
class User {
  final int id;
  final String name;
  final String email;

  const User({required this.id, required this.name, required this.email});

  factory User.fromJson(Map<String, dynamic> json) => User(
    id: json['id'] as int,
    name: json['name'] as String,
    email: json['email'] as String,
  );

  Map<String, dynamic> toJson() => {'id': id, 'name': name, 'email': email};
}

// Con Dio — cliente más robusto
import 'package:dio/dio.dart';

final dio = Dio(BaseOptions(
  baseUrl: 'https://api.example.com',
  connectTimeout: const Duration(seconds: 5),
  receiveTimeout: const Duration(seconds: 10),
  headers: {'Content-Type': 'application/json'},
));

// Interceptor para auth
dio.interceptors.add(InterceptorsWrapper(
  onRequest: (options, handler) {
    options.headers['Authorization'] = 'Bearer $token';
    handler.next(options);
  },
  onError: (error, handler) {
    if (error.response?.statusCode == 401) {
      // Refresh token
    }
    handler.next(error);
  },
));

final response = await dio.get('/users');
final users = (response.data as List).map((e) => User.fromJson(e)).toList();
```

---

## 💾 Persistencia local

```yaml
dependencies:
  shared_preferences: ^2.2.0  # Clave-valor simple
  sqflite: ^2.3.0             # SQLite
  hive_flutter: ^1.1.0        # NoSQL rápido
  isar: ^3.1.0                # Base de datos moderna con Flutter
```

```dart
// Shared Preferences
import 'package:shared_preferences/shared_preferences.dart';

// Guardar
final prefs = await SharedPreferences.getInstance();
await prefs.setString('token', 'abc123');
await prefs.setInt('userId', 42);
await prefs.setBool('isDarkMode', true);

// Leer
final token = prefs.getString('token');
final userId = prefs.getInt('userId') ?? 0;
final isDark = prefs.getBool('isDarkMode') ?? false;

// Eliminar
await prefs.remove('token');
await prefs.clear();
```

---

## 🎬 Animaciones

```dart
// AnimatedContainer — anima automáticamente cambios de propiedades
AnimatedContainer(
  duration: const Duration(milliseconds: 300),
  curve: Curves.easeInOut,
  width: _expanded ? 200 : 100,
  color: _expanded ? Colors.blue : Colors.red,
  child: const Text('Hola'),
)

// AnimatedOpacity
AnimatedOpacity(
  duration: const Duration(milliseconds: 200),
  opacity: _visible ? 1.0 : 0.0,
  child: const Text('Aparezco'),
)

// AnimatedSwitcher — anima cambio entre widgets
AnimatedSwitcher(
  duration: const Duration(milliseconds: 300),
  child: Text('$_count', key: ValueKey(_count)),   // key es crucial
)

// Hero — animación entre pantallas
// Pantalla 1:
Hero(tag: 'product-${item.id}', child: Image.network(item.imageUrl))
// Pantalla 2 (misma tag):
Hero(tag: 'product-${item.id}', child: Image.network(item.imageUrl, height: 300))

// TweenAnimationBuilder — animación simple con valor
TweenAnimationBuilder<double>(
  tween: Tween(begin: 0, end: 1),
  duration: const Duration(seconds: 1),
  builder: (context, value, child) => Opacity(opacity: value, child: child),
  child: const Text('Fade in'),
)
```

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

---

## 📦 Paquetes esenciales

| Categoría | Paquete | Descripción |
|---|---|---|
| Estado | `provider` | Gestión de estado simple y escalable |
| Estado | `flutter_riverpod` | Gestión de estado moderna y robusta |
| Estado | `bloc` + `flutter_bloc` | Patrón BLoC estricto |
| Navegación | `go_router` | Router declarativo oficial |
| HTTP | `dio` | Cliente HTTP con interceptores |
| HTTP | `http` | Cliente HTTP simple |
| Serialización | `json_serializable` | Generación de fromJson/toJson |
| Serialización | `freezed` | Clases inmutables + unions |
| Persistencia | `shared_preferences` | Clave-valor simple |
| Persistencia | `hive_flutter` | Base de datos NoSQL local rápida |
| Persistencia | `sqflite` | SQLite para Flutter |
| Inyección de dep. | `get_it` | Service locator |
| Imágenes | `cached_network_image` | Caché de imágenes de red |
| Imágenes | `image_picker` | Seleccionar imagen de galería/cámara |
| Auth | `firebase_auth` | Autenticación con Firebase |
| Push | `firebase_messaging` | Notificaciones push |
| Forms | `reactive_forms` | Manejo avanzado de formularios |
| UI | `flutter_svg` | Soporte de SVG |
| UI | `shimmer` | Efecto shimmer de carga |
| UI | `lottie` | Animaciones Lottie |
| Testing | `mocktail` | Mocking para tests |
| Logging | `logger` | Logging con niveles y estilos |
| Env | `flutter_dotenv` | Variables de entorno desde `.env` |

---

## 🧪 Testing

```dart
// Unit test
import 'package:flutter_test/flutter_test.dart';

void main() {
  group('CounterProvider', () {
    late CounterProvider counter;

    setUp(() => counter = CounterProvider());

    test('empieza en 0', () {
      expect(counter.count, 0);
    });

    test('incrementa correctamente', () {
      counter.increment();
      expect(counter.count, 1);
    });
  });
}

// Widget test
testWidgets('muestra el título', (WidgetTester tester) async {
  await tester.pumpWidget(const MaterialApp(home: HomePage()));
  expect(find.text('Inicio'), findsOneWidget);
  expect(find.byType(FloatingActionButton), findsOneWidget);

  await tester.tap(find.byType(FloatingActionButton));
  await tester.pump();   // Reconstruir tras setState
});
```

---

## 💡 Buenas prácticas

- **Preferir `const` siempre que sea posible** — reduce reconstrucciones innecesarias.
- **Extraer widgets en lugar de anidar** — métodos privados que devuelven Widget son válidos, pero widgets separados son más reutilizables y mejoran el rebuild.
- **No poner lógica en el `build()`** — moverla a providers, use cases o controllers.
- **Siempre disponer los controllers** (`TextEditingController`, `AnimationController`, etc.) en `dispose()`.
- **Usar `ListView.builder` para listas largas** — nunca `ListView` con children fijos para listas dinámicas.
- **Nombrar `key` en listas dinámicas** — `ValueKey`, `ObjectKey` para que Flutter identifique elementos correctamente.
- **Separar la UI de la lógica** — Feature Folder o Clean Architecture según la escala del proyecto.
- **`context.read()` para acciones, `context.watch()` para estado** — no usar `watch` fuera del `build`.
- **Testear la lógica de negocio** con unit tests, no la UI en primer lugar.
- **Usar `flutter analyze` y `dart fix`** regularmente para mantener el código limpio.

---

## 🎯 Dart moderno — Dart 3.x

### Null safety

```dart
// Tipos nullable vs non-nullable
String name = "Juan";          // Non-nullable — nunca puede ser null
String? nickname = null;       // Nullable — puede ser null

// Operadores null safety
String display = nickname ?? "Sin apodo";        // ?? — valor por defecto si null
int? length = nickname?.length;                  // ?. — acceso seguro (null si es null)
String forced = nickname!;                       // ! — aserción (lanza si es null, evitar)
nickname ??= "Default";                          // ??= — asignar solo si es null

// late — inicialización diferida (no null pero se asigna después)
late String userId;             // Se asignará antes de usarse
late final String token;        // Asignación única, luego inmutable

// required — parámetros nombrados obligatorios
class UserCard extends StatelessWidget {
  const UserCard({
    super.key,
    required this.name,         // Obligatorio
    this.avatarUrl,             // Opcional (nullable)
    this.onTap,                 // Opcional
  });

  final String name;
  final String? avatarUrl;
  final VoidCallback? onTap;
}
```

---

### Records y destructuring (Dart 3+)

```dart
// Records — tuplas con tipo estático
(String, int) getUserInfo() => ("Juan", 28);

final (name, age) = getUserInfo();   // Destructuring
print("$name tiene $age años");

// Records con campos nombrados
({String name, int age, String email}) getUser() =>
    (name: "Juan", age: 28, email: "juan@email.com");

final user = getUser();
print(user.name);    // Acceso por nombre
print(user.age);

// Destructuring en múltiples variables
final (:name, :age, :email) = getUser();

// En listas y mapas
final [first, second, ...rest] = [1, 2, 3, 4, 5];
final {"key1": v1, "key2": v2} = {"key1": "a", "key2": "b"};
```

---

### Pattern matching y sealed classes (Dart 3+)

```dart
// sealed class — jerarquía cerrada, el compilador sabe todos los subtipos
sealed class AuthState {}
class Unauthenticated extends AuthState {}
class Authenticated extends AuthState {
  final String userId;
  final String email;
  Authenticated({required this.userId, required this.email});
}
class AuthLoading extends AuthState {}
class AuthError extends AuthState {
  final String message;
  AuthError(this.message);
}

// switch exhaustivo — el compilador exige todos los casos
String describeAuth(AuthState state) => switch (state) {
  Unauthenticated() => "No autenticado",
  Authenticated(:final email) => "Autenticado como $email",
  AuthLoading() => "Cargando...",
  AuthError(:final message) => "Error: $message",
};

// Pattern matching con if-case
void handleState(AuthState state) {
  if (state case Authenticated(:final userId, :final email)) {
    print("User $userId: $email");
  }
}

// Switch con guards
String classify(int n) => switch (n) {
  0 => "cero",
  int x when x < 0 => "negativo",
  int x when x.isEven => "positivo par",
  _ => "positivo impar",
};

// sealed class para resultados (reemplaza Either)
sealed class Result<T> {}
class Success<T> extends Result<T> {
  final T data;
  Success(this.data);
}
class Failure<T> extends Result<T> {
  final String message;
  final Object? error;
  Failure(this.message, {this.error});
}

// Uso en repositorio
Future<Result<List<Product>>> fetchProducts() async {
  try {
    final data = await api.getProducts();
    return Success(data.map((e) => e.toDomain()).toList());
  } on DioException catch (e) {
    return Failure("Error de red: ${e.message}");
  } catch (e) {
    return Failure("Error inesperado", error: e);
  }
}

// Consumo exhaustivo
switch (await fetchProducts()) {
  case Success(:final data): showProducts(data);
  case Failure(:final message): showError(message);
}
```

---

### Extensions, Mixins y Generics

```dart
// Extension methods — agregar funcionalidad a tipos existentes
extension StringExtensions on String {
  String capitalize() =>
      isEmpty ? this : "${this[0].toUpperCase()}${substring(1)}";

  bool get isValidEmail =>
      RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(this);

  String truncate(int maxLength, {String ellipsis = "..."}) =>
      length <= maxLength ? this : "${substring(0, maxLength)}$ellipsis";
}

extension DateTimeExtensions on DateTime {
  bool get isToday {
    final now = DateTime.now();
    return day == now.day && month == now.month && year == now.year;
  }
  String get timeAgo {
    final diff = DateTime.now().difference(this);
    if (diff.inDays > 0) return "hace ${diff.inDays}d";
    if (diff.inHours > 0) return "hace ${diff.inHours}h";
    return "hace ${diff.inMinutes}m";
  }
}

extension ListExtensions<T> on List<T> {
  List<T> get nonNulls => whereType<T>().toList();
  T? firstWhereOrNull(bool Function(T) test) {
    for (final e in this) if (test(e)) return e;
    return null;
  }
}

// Uso
"hola mundo".capitalize();     // "Hola mundo"
"test@email.com".isValidEmail; // true
DateTime.now().isToday;        // true

// Mixins — reutilizar comportamiento sin herencia
mixin LoadingMixin {
  bool _isLoading = false;
  bool get isLoading => _isLoading;

  void setLoading(bool value) {
    _isLoading = value;
  }
}

mixin ValidationMixin {
  String? validateEmail(String? value) {
    if (value == null || value.isEmpty) return "Campo requerido";
    if (!value.isValidEmail) return "Email inválido";
    return null;
  }
  String? validatePassword(String? value) {
    if (value == null || value.length < 8) return "Mínimo 8 caracteres";
    return null;
  }
}

class AuthViewModel extends ChangeNotifier with LoadingMixin, ValidationMixin {
  Future<void> login(String email, String password) async {
    setLoading(true);
    notifyListeners();
    // ...
    setLoading(false);
    notifyListeners();
  }
}

// Generics
class Repository<T> {
  final Map<String, T> _cache = {};

  T? getFromCache(String key) => _cache[key];
  void saveToCache(String key, T value) => _cache[key] = value;

  Future<T> fetchById(String id, Future<T> Function(String) fetcher) async {
    final cached = getFromCache(id);
    if (cached != null) return cached;
    final result = await fetcher(id);
    saveToCache(id, result);
    return result;
  }
}

// typedef — alias de tipos
typedef JsonMap = Map<String, dynamic>;
typedef VoidAsyncCallback = Future<void> Function();
typedef ProductMapper = Product Function(JsonMap json);
```

---

### Modelos con freezed + json_serializable

```yaml
# pubspec.yaml
dependencies:
  freezed_annotation: ^2.4.4
  json_annotation: ^4.9.0

dev_dependencies:
  freezed: ^2.5.7
  json_serializable: ^6.8.0
  build_runner: ^2.4.13
```

```dart
// product.dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'product.freezed.dart';
part 'product.g.dart';

@freezed
class Product with _$Product {
  const factory Product({
    required String id,
    required String name,
    required double price,
    @JsonKey(name: 'image_url') required String imageUrl,
    String? category,
    @Default([]) List<String> tags,
    @Default(false) bool isFavorite,
    DateTime? createdAt,
  }) = _Product;

  factory Product.fromJson(JsonMap json) => _$ProductFromJson(json);
}

// Uso — copyWith, ==, hashCode, toString generados automáticamente
final product = Product(id: "1", name: "Laptop", price: 999.99, imageUrl: "...");
final updated = product.copyWith(price: 899.99, isFavorite: true);
final json = product.toJson();
final fromJson = Product.fromJson(json);

// Union types con freezed (equivalente a sealed class)
@freezed
sealed class AuthState with _$AuthState {
  const factory AuthState.initial() = AuthInitial;
  const factory AuthState.loading() = AuthLoading;
  const factory AuthState.authenticated({required User user}) = AuthAuthenticated;
  const factory AuthState.unauthenticated() = AuthUnauthenticated;
  const factory AuthState.error({required String message}) = AuthError;
}

// Uso con map/when
state.when(
  initial: () => const SplashScreen(),
  loading: () => const LoadingIndicator(),
  authenticated: (user) => HomeScreen(user: user),
  unauthenticated: () => const LoginScreen(),
  error: (message) => ErrorScreen(message: message),
);

// maybeWhen — solo manejar algunos casos
state.maybeWhen(
  authenticated: (user) => Text(user.name),
  orElse: () => const SizedBox.shrink(),
);

// Generar código
// flutter pub run build_runner build --delete-conflicting-outputs
// flutter pub run build_runner watch  (modo watch durante desarrollo)
```

---

## ⚡ Riverpod — completo con code generation

```yaml
dependencies:
  flutter_riverpod: ^2.6.1
  riverpod_annotation: ^2.6.1

dev_dependencies:
  riverpod_generator: ^2.6.1
  build_runner: ^2.4.13
```

```dart
// main.dart
void main() {
  runApp(const ProviderScope(child: MyApp()));
}
```

### Providers con @riverpod

```dart
// providers.dart
part 'providers.g.dart';

// Provider simple — valor inmutable
@riverpod
String appVersion(AppVersionRef ref) => "1.0.0";

// Provider con dependencia de otro provider
@riverpod
ApiService apiService(ApiServiceRef ref) {
  final baseUrl = ref.watch(appConfigProvider).apiBaseUrl;
  return ApiService(baseUrl: baseUrl);
}

// FutureProvider — datos async
@riverpod
Future<List<Product>> products(ProductsRef ref) async {
  final repo = ref.watch(productRepositoryProvider);
  return repo.getAll();
}

// FutureProvider con parámetro (family)
@riverpod
Future<Product> productDetail(ProductDetailRef ref, String id) async {
  return ref.watch(productRepositoryProvider).getById(id);
}

// StreamProvider — datos en tiempo real
@riverpod
Stream<List<Product>> productsStream(ProductsStreamRef ref) {
  return ref.watch(productRepositoryProvider).watchAll();
}

// Mantener vivo aunque no haya suscriptores
@Riverpod(keepAlive: true)
UserSession userSession(UserSessionRef ref) => UserSession();
```

---

### AsyncNotifier y Notifier

```dart
// AsyncNotifier — para estado async con operaciones de escritura
@riverpod
class ProductsNotifier extends _$ProductsNotifier {
  @override
  Future<List<Product>> build() async {
    // Se llama automáticamente al crear el provider
    return ref.watch(productRepositoryProvider).getAll();
  }

  Future<void> addProduct(CreateProductRequest request) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      final repo = ref.read(productRepositoryProvider);
      await repo.create(request);
      return repo.getAll();      // Recargar lista completa
    });
  }

  Future<void> deleteProduct(String id) async {
    final previous = state;
    // Optimistic update — actualizar UI antes de confirmar con servidor
    state = AsyncData(
      state.value?.where((p) => p.id != id).toList() ?? [],
    );
    try {
      await ref.read(productRepositoryProvider).delete(id);
      ref.invalidateSelf();      // Recargar desde servidor
    } catch (e) {
      state = previous;          // Revertir si falla
    }
  }

  void invalidate() => ref.invalidateSelf();
}

// Notifier — para estado síncrono con operaciones
@riverpod
class CartNotifier extends _$CartNotifier {
  @override
  List<CartItem> build() => [];

  void addItem(Product product) {
    final existing = state.firstWhereOrNull((i) => i.product.id == product.id);
    if (existing != null) {
      state = state.map((i) =>
        i.product.id == product.id ? i.copyWith(quantity: i.quantity + 1) : i
      ).toList();
    } else {
      state = [...state, CartItem(product: product, quantity: 1)];
    }
  }

  void removeItem(String productId) {
    state = state.where((i) => i.product.id != productId).toList();
  }

  void clear() => state = [];
}

// StreamNotifier — para estado basado en Stream
@riverpod
class AuthNotifier extends _$AuthNotifier {
  @override
  Stream<AuthState> build() {
    return ref.watch(authRepositoryProvider).authStateChanges;
  }

  Future<void> signIn(String email, String password) async {
    await ref.read(authRepositoryProvider).signIn(email, password);
  }

  Future<void> signOut() async {
    await ref.read(authRepositoryProvider).signOut();
  }
}
```

---

### Consumir providers en widgets

```dart
// ConsumerWidget — Stateless con acceso a Riverpod
class ProductsScreen extends ConsumerWidget {
  const ProductsScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final productsAsync = ref.watch(productsNotifierProvider);

    return Scaffold(
      body: productsAsync.when(
        data: (products) => ProductsList(products: products),
        loading: () => const CircularProgressIndicator(),
        error: (error, stack) => ErrorView(
          message: error.toString(),
          onRetry: () => ref.invalidate(productsNotifierProvider),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => ref.read(productsNotifierProvider.notifier).invalidate(),
        child: const Icon(Icons.refresh),
      ),
    );
  }
}

// ConsumerStatefulWidget — Stateful con acceso a Riverpod
class SearchScreen extends ConsumerStatefulWidget {
  const SearchScreen({super.key});

  @override
  ConsumerState<SearchScreen> createState() => _SearchScreenState();
}

class _SearchScreenState extends ConsumerState<SearchScreen> {
  final _controller = TextEditingController();

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final results = ref.watch(searchResultsProvider(_controller.text));
    // ...
  }
}

// ref.watch  → suscribirse, reconstruye cuando cambia
// ref.read   → leer una vez sin suscripción (en callbacks/handlers)
// ref.listen → reaccionar sin reconstruir (para side effects)
// ref.invalidate → forzar recálculo del provider

// ref.listen — para side effects (snackbars, navegación)
@override
Widget build(BuildContext context, WidgetRef ref) {
  ref.listen<AsyncValue<void>>(
    authNotifierProvider,
    (previous, next) {
      next.whenOrNull(
        error: (error, _) => ScaffoldMessenger.of(context)
            .showSnackBar(SnackBar(content: Text(error.toString()))),
      );
    },
  );
  // ...
}

// select — suscribirse solo a parte del estado (optimización)
final userName = ref.watch(userProvider.select((u) => u.name));
// Solo reconstruye si user.name cambia, no otros campos

// Providers con familia en widget
class ProductDetailScreen extends ConsumerWidget {
  final String productId;
  const ProductDetailScreen({super.key, required this.productId});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final productAsync = ref.watch(productDetailProvider(productId));
    // ...
  }
}
```

---

## 🧱 BLoC Pattern

```yaml
dependencies:
  flutter_bloc: ^8.1.6
  bloc: ^8.1.4
```

```dart
// Cubit — para lógica simple sin eventos explícitos
class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);

  void increment() => emit(state + 1);
  void decrement() => emit(state - 1);
  void reset() => emit(0);
}

// Cubit con estado complejo
class ProductsCubit extends Cubit<ProductsState> {
  ProductsCubit(this._repository) : super(const ProductsState.initial());

  final ProductRepository _repository;

  Future<void> loadProducts() async {
    emit(const ProductsState.loading());
    switch (await _repository.getAll()) {
      case Success(:final data):
        emit(ProductsState.loaded(products: data));
      case Failure(:final message):
        emit(ProductsState.error(message: message));
    }
  }

  void toggleFavorite(String productId) {
    if (state case ProductsLoaded(:final products)) {
      final updated = products.map((p) =>
        p.id == productId ? p.copyWith(isFavorite: !p.isFavorite) : p
      ).toList();
      emit(ProductsState.loaded(products: updated));
    }
  }
}

// BLoC — para lógica compleja con eventos explícitos
// Eventos
sealed class AuthEvent {}
class AuthLoginRequested extends AuthEvent {
  final String email;
  final String password;
  AuthLoginRequested({required this.email, required this.password});
}
class AuthLogoutRequested extends AuthEvent {}
class AuthGoogleSignInRequested extends AuthEvent {}

// BLoC
class AuthBloc extends Bloc<AuthEvent, AuthState> {
  AuthBloc(this._authRepository) : super(const AuthState.initial()) {
    on<AuthLoginRequested>(_onLoginRequested);
    on<AuthLogoutRequested>(_onLogoutRequested);
    on<AuthGoogleSignInRequested>(_onGoogleSignIn);
  }

  final AuthRepository _authRepository;

  Future<void> _onLoginRequested(
    AuthLoginRequested event,
    Emitter<AuthState> emit,
  ) async {
    emit(const AuthState.loading());
    switch (await _authRepository.signIn(event.email, event.password)) {
      case Success(:final data):
        emit(AuthState.authenticated(user: data));
      case Failure(:final message):
        emit(AuthState.error(message: message));
    }
  }

  Future<void> _onLogoutRequested(
    AuthLogoutRequested event,
    Emitter<AuthState> emit,
  ) async {
    await _authRepository.signOut();
    emit(const AuthState.unauthenticated());
  }

  Future<void> _onGoogleSignIn(
    AuthGoogleSignInRequested event,
    Emitter<AuthState> emit,
  ) async {
    emit(const AuthState.loading());
    switch (await _authRepository.signInWithGoogle()) {
      case Success(:final data): emit(AuthState.authenticated(user: data));
      case Failure(:final message): emit(AuthState.error(message: message));
    }
  }
}
```

---

### BLoC en la UI

```dart
// Proveer BLoC/Cubit
BlocProvider(
  create: (context) => ProductsCubit(context.read<ProductRepository>())..loadProducts(),
  child: const ProductsScreen(),
)

// Múltiples providers
MultiBlocProvider(
  providers: [
    BlocProvider(create: (_) => sl<AuthBloc>()),
    BlocProvider(create: (_) => sl<CartCubit>()),
  ],
  child: const MyApp(),
)

// BlocBuilder — reconstruir UI cuando cambia el estado
BlocBuilder<ProductsCubit, ProductsState>(
  // buildWhen — evitar reconstrucciones innecesarias
  buildWhen: (previous, current) => previous != current,
  builder: (context, state) => switch (state) {
    ProductsInitial() => const SizedBox.shrink(),
    ProductsLoading() => const CircularProgressIndicator(),
    ProductsLoaded(:final products) => ProductsList(products: products),
    ProductsError(:final message) => ErrorView(message: message),
  },
)

// BlocListener — side effects sin reconstruir UI
BlocListener<AuthBloc, AuthState>(
  listener: (context, state) {
    if (state case AuthError(:final message)) {
      ScaffoldMessenger.of(context)
          .showSnackBar(SnackBar(content: Text(message)));
    }
    if (state case AuthAuthenticated()) {
      context.go('/home');
    }
  },
  child: const LoginForm(),
)

// BlocConsumer — ambos a la vez
BlocConsumer<AuthBloc, AuthState>(
  listener: (context, state) {
    if (state case AuthAuthenticated()) context.go('/home');
  },
  builder: (context, state) => switch (state) {
    AuthLoading() => const CircularProgressIndicator(),
    _ => LoginForm(
        onSubmit: (email, pass) => context.read<AuthBloc>()
            .add(AuthLoginRequested(email: email, password: pass)),
      ),
  },
)

// context.read  → acceder sin suscribirse (en callbacks)
// context.watch → suscribirse y reconstruir
// context.select → suscribirse a parte del estado
final isLoading = context.select<AuthBloc, bool>(
  (bloc) => bloc.state is AuthLoading,
);
```

---

## 🌐 Dio — HTTP completo

```yaml
dependencies:
  dio: ^5.7.0
  pretty_dio_logger: ^1.4.0
```

```dart
// dio_client.dart
class DioClient {
  late final Dio _dio;

  DioClient({
    required String baseUrl,
    required TokenStorage tokenStorage,
    required AuthRepository authRepository,
  }) {
    _dio = Dio(BaseOptions(
      baseUrl: baseUrl,
      connectTimeout: const Duration(seconds: 30),
      receiveTimeout: const Duration(seconds: 30),
      sendTimeout: const Duration(seconds: 60),
      headers: {'Content-Type': 'application/json', 'Accept': 'application/json'},
    ));

    _dio.interceptors.addAll([
      AuthInterceptor(tokenStorage: tokenStorage, authRepository: authRepository, dio: _dio),
      if (kDebugMode) PrettyDioLogger(requestBody: true, responseBody: true),
      RetryInterceptor(dio: _dio, retries: 3),
    ]);
  }

  Dio get instance => _dio;
}

// AuthInterceptor con refresh token
class AuthInterceptor extends Interceptor {
  AuthInterceptor({
    required this.tokenStorage,
    required this.authRepository,
    required this.dio,
  });

  final TokenStorage tokenStorage;
  final AuthRepository authRepository;
  final Dio dio;
  bool _isRefreshing = false;
  final _queue = <_RetryRequest>[];

  @override
  Future<void> onRequest(
    RequestOptions options,
    RequestInterceptorHandler handler,
  ) async {
    final token = await tokenStorage.getAccessToken();
    if (token != null) {
      options.headers['Authorization'] = 'Bearer $token';
    }
    handler.next(options);
  }

  @override
  Future<void> onError(
    DioException err,
    ErrorInterceptorHandler handler,
  ) async {
    if (err.response?.statusCode != 401) {
      handler.next(err);
      return;
    }

    if (_isRefreshing) {
      // Encolar la request para reintentarla después del refresh
      final completer = Completer<Response>();
      _queue.add(_RetryRequest(err.requestOptions, completer));
      handler.resolve(await completer.future);
      return;
    }

    _isRefreshing = true;
    try {
      final newToken = await authRepository.refreshToken();
      await tokenStorage.saveAccessToken(newToken);

      // Reintentar la request original
      final response = await dio.fetch(
        err.requestOptions..headers['Authorization'] = 'Bearer $newToken',
      );

      // Reintentar requests encoladas
      for (final req in _queue) {
        req.completer.complete(
          await dio.fetch(req.options..headers['Authorization'] = 'Bearer $newToken'),
        );
      }
      _queue.clear();
      handler.resolve(response);
    } catch (e) {
      _queue.clear();
      await authRepository.signOut();
      handler.next(err);
    } finally {
      _isRefreshing = false;
    }
  }
}

class _RetryRequest {
  final RequestOptions options;
  final Completer<Response> completer;
  _RetryRequest(this.options, this.completer);
}

// API Service
class ProductApiService {
  ProductApiService(this._dio);
  final Dio _dio;

  Future<List<ProductDto>> getProducts({
    String? category,
    int page = 1,
    int limit = 20,
  }) async {
    final response = await _dio.get(
      '/products',
      queryParameters: {
        if (category != null) 'category': category,
        'page': page,
        'limit': limit,
      },
    );
    return (response.data as List)
        .map((e) => ProductDto.fromJson(e as JsonMap))
        .toList();
  }

  Future<ProductDto> createProduct(CreateProductRequest request) async {
    final response = await _dio.post('/products', data: request.toJson());
    return ProductDto.fromJson(response.data as JsonMap);
  }

  Future<ProductDto> updateProduct(String id, UpdateProductRequest request) async {
    final response = await _dio.patch('/products/$id', data: request.toJson());
    return ProductDto.fromJson(response.data as JsonMap);
  }

  Future<void> deleteProduct(String id) => _dio.delete('/products/$id');

  // Subir archivo con progreso
  Future<String> uploadImage(
    File imageFile, {
    required void Function(int sent, int total) onProgress,
  }) async {
    final formData = FormData.fromMap({
      'image': await MultipartFile.fromFile(
        imageFile.path,
        filename: path.basename(imageFile.path),
        contentType: DioMediaType('image', 'jpeg'),
      ),
    });

    final response = await _dio.post(
      '/upload',
      data: formData,
      onSendProgress: onProgress,
    );
    return response.data['url'] as String;
  }

  // Descargar archivo con progreso
  Future<void> downloadFile(
    String url,
    String savePath, {
    void Function(int received, int total)? onProgress,
  }) async {
    await _dio.download(url, savePath, onReceiveProgress: onProgress);
  }

  // Cancelar request
  Future<List<ProductDto>> searchWithCancel(String query) async {
    final cancelToken = CancelToken();
    try {
      final response = await _dio.get(
        '/products/search',
        queryParameters: {'q': query},
        cancelToken: cancelToken,
      );
      return (response.data as List)
          .map((e) => ProductDto.fromJson(e as JsonMap))
          .toList();
    } on DioException catch (e) {
      if (CancelToken.isCancel(e)) return [];
      rethrow;
    }
  }
}

// Manejo de errores centralizado
class ApiException implements Exception {
  final String message;
  final int? statusCode;
  const ApiException(this.message, {this.statusCode});
}

DioException toDomain(DioException e) => e; // para rethrow

Future<T> safeCall<T>(Future<T> Function() call) async {
  try {
    return await call();
  } on DioException catch (e) {
    throw switch (e.type) {
      DioExceptionType.connectionTimeout ||
      DioExceptionType.receiveTimeout ||
      DioExceptionType.sendTimeout =>
          ApiException("Tiempo de espera agotado"),
      DioExceptionType.connectionError =>
          ApiException("Sin conexión a internet"),
      DioExceptionType.badResponse => ApiException(
          e.response?.data?['message'] as String? ?? "Error del servidor",
          statusCode: e.response?.statusCode,
        ),
      _ => ApiException("Error inesperado: ${e.message}"),
    };
  }
}
```

---

## 🔥 Firebase completo

```yaml
dependencies:
  firebase_core: ^3.8.0
  firebase_auth: ^5.3.3
  cloud_firestore: ^5.5.0
  firebase_storage: ^12.3.5
  firebase_messaging: ^15.1.5
  firebase_crashlytics: ^4.1.4
  firebase_analytics: ^11.3.5
  google_sign_in: ^6.2.2
  sign_in_with_apple: ^6.1.2
```

```dart
// main.dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform);

  // Crashlytics
  FlutterError.onError = FirebaseCrashlytics.instance.recordFlutterFatalError;
  PlatformDispatcher.instance.onError = (error, stack) {
    FirebaseCrashlytics.instance.recordError(error, stack, fatal: true);
    return true;
  };

  runApp(const ProviderScope(child: MyApp()));
}
```

---

### Firebase Auth

```dart
class AuthRepository {
  final _auth = FirebaseAuth.instance;

  Stream<User?> get authStateChanges => _auth.authStateChanges();
  User? get currentUser => _auth.currentUser;
  bool get isLoggedIn => currentUser != null;

  Future<Result<User>> signIn(String email, String password) async {
    try {
      final cred = await _auth.signInWithEmailAndPassword(
          email: email, password: password);
      return Success(cred.user!);
    } on FirebaseAuthException catch (e) {
      return Failure(_authErrorMessage(e.code));
    }
  }

  Future<Result<User>> register(String email, String password, String name) async {
    try {
      final cred = await _auth.createUserWithEmailAndPassword(
          email: email, password: password);
      await cred.user!.updateDisplayName(name);
      await cred.user!.sendEmailVerification();
      return Success(cred.user!);
    } on FirebaseAuthException catch (e) {
      return Failure(_authErrorMessage(e.code));
    }
  }

  // Google Sign-In
  Future<Result<User>> signInWithGoogle() async {
    try {
      final googleUser = await GoogleSignIn().signIn();
      if (googleUser == null) return Failure("Inicio cancelado");

      final googleAuth = await googleUser.authentication;
      final credential = GoogleAuthProvider.credential(
        accessToken: googleAuth.accessToken,
        idToken: googleAuth.idToken,
      );
      final cred = await _auth.signInWithCredential(credential);
      return Success(cred.user!);
    } catch (e) {
      return Failure("Error al iniciar con Google");
    }
  }

  // Apple Sign-In (obligatorio en iOS si ofrecés login social)
  Future<Result<User>> signInWithApple() async {
    try {
      final appleCredential = await SignInWithApple.getAppleIDCredential(
        scopes: [AppleIDAuthorizationScopes.email, AppleIDAuthorizationScopes.fullName],
      );
      final oauthCredential = OAuthProvider("apple.com").credential(
        idToken: appleCredential.identityToken,
        accessToken: appleCredential.authorizationCode,
      );
      final cred = await _auth.signInWithCredential(oauthCredential);
      return Success(cred.user!);
    } catch (e) {
      return Failure("Error al iniciar con Apple");
    }
  }

  Future<void> signOut() async {
    await GoogleSignIn().signOut();
    await _auth.signOut();
  }

  Future<Result<void>> sendPasswordReset(String email) async {
    try {
      await _auth.sendPasswordResetEmail(email: email);
      return Success(null);
    } on FirebaseAuthException catch (e) {
      return Failure(_authErrorMessage(e.code));
    }
  }

  Future<String?> getIdToken({bool forceRefresh = false}) =>
      currentUser?.getIdToken(forceRefresh);

  String _authErrorMessage(String code) => switch (code) {
    'user-not-found' => "No existe una cuenta con este email",
    'wrong-password' => "Contraseña incorrecta",
    'email-already-in-use' => "Este email ya está en uso",
    'weak-password' => "La contraseña debe tener al menos 6 caracteres",
    'invalid-email' => "Email inválido",
    'too-many-requests' => "Demasiados intentos. Intentá más tarde",
    'network-request-failed' => "Error de conexión",
    _ => "Error de autenticación: $code",
  };
}
```

---

### Firestore CRUD + streams

```dart
class ProductFirestoreRepository {
  final _db = FirebaseFirestore.instance;
  CollectionReference get _col => _db.collection('products');

  // Crear con ID automático
  Future<Result<String>> create(Product product) async {
    try {
      final ref = await _col.add(product.toFirestoreMap());
      return Success(ref.id);
    } catch (e) {
      return Failure("Error al crear: $e");
    }
  }

  // Leer por ID
  Future<Result<Product?>> getById(String id) async {
    try {
      final doc = await _col.doc(id).get();
      return Success(doc.exists ? Product.fromFirestore(doc) : null);
    } catch (e) {
      return Failure("Error al leer: $e");
    }
  }

  // Stream en tiempo real de toda la colección
  Stream<List<Product>> watchAll({String? category}) {
    Query query = _col.orderBy('createdAt', descending: true);
    if (category != null) query = query.where('category', isEqualTo: category);

    return query.snapshots().map(
      (snapshot) => snapshot.docs
          .map((doc) => Product.fromFirestore(doc))
          .toList(),
    );
  }

  // Stream de documento individual
  Stream<Product?> watchById(String id) => _col.doc(id)
      .snapshots()
      .map((doc) => doc.exists ? Product.fromFirestore(doc) : null);

  // Actualizar campos específicos
  Future<Result<void>> update(String id, Map<String, dynamic> fields) async {
    try {
      await _col.doc(id).update({
        ...fields,
        'updatedAt': FieldValue.serverTimestamp(),
      });
      return Success(null);
    } catch (e) {
      return Failure("Error al actualizar: $e");
    }
  }

  // Eliminar
  Future<Result<void>> delete(String id) async {
    try {
      await _col.doc(id).delete();
      return Success(null);
    } catch (e) {
      return Failure("Error al eliminar: $e");
    }
  }

  // Paginación con cursor
  Future<Result<({List<Product> items, DocumentSnapshot? lastDoc})>> getPage({
    DocumentSnapshot? startAfter,
    int limit = 20,
    String? category,
  }) async {
    try {
      Query query = _col
          .orderBy('createdAt', descending: true)
          .limit(limit);
      if (category != null) query = query.where('category', isEqualTo: category);
      if (startAfter != null) query = query.startAfterDocument(startAfter);

      final snapshot = await query.get();
      final items = snapshot.docs.map(Product.fromFirestore).toList();
      final lastDoc = snapshot.docs.lastOrNull;
      return Success((items: items, lastDoc: lastDoc));
    } catch (e) {
      return Failure("Error de paginación: $e");
    }
  }

  // Transacción atómica
  Future<Result<void>> purchaseProduct(String productId, int quantity) async {
    try {
      await _db.runTransaction((transaction) async {
        final ref = _col.doc(productId);
        final doc = await transaction.get(ref);
        final stock = (doc.data() as JsonMap)['stock'] as int;

        if (stock < quantity) throw Exception("Stock insuficiente");

        transaction.update(ref, {'stock': stock - quantity});
        transaction.set(_db.collection('orders').doc(), {
          'productId': productId,
          'quantity': quantity,
          'createdAt': FieldValue.serverTimestamp(),
        });
      });
      return Success(null);
    } catch (e) {
      return Failure(e.toString());
    }
  }
}

// Mappers
extension ProductFirestoreX on Product {
  Map<String, dynamic> toFirestoreMap() => {
    'name': name,
    'price': price,
    'imageUrl': imageUrl,
    'category': category,
    'tags': tags,
    'createdAt': FieldValue.serverTimestamp(),
  };

  static Product fromFirestore(DocumentSnapshot doc) {
    final data = doc.data() as JsonMap;
    return Product(
      id: doc.id,
      name: data['name'] as String,
      price: (data['price'] as num).toDouble(),
      imageUrl: data['imageUrl'] as String,
      category: data['category'] as String?,
      tags: List<String>.from(data['tags'] as List? ?? []),
    );
  }
}
```

---

### Firebase Storage

```dart
class StorageRepository {
  final _storage = FirebaseStorage.instance;

  Future<Result<String>> uploadFile({
    required File file,
    required String path,
    String contentType = 'image/jpeg',
    void Function(double progress)? onProgress,
  }) async {
    try {
      final ref = _storage.ref(path);
      final metadata = SettableMetadata(contentType: contentType);
      final task = ref.putFile(file, metadata);

      if (onProgress != null) {
        task.snapshotEvents.listen((snapshot) {
          final progress = snapshot.bytesTransferred / snapshot.totalBytes;
          onProgress(progress);
        });
      }

      await task;
      return Success(await ref.getDownloadURL());
    } catch (e) {
      return Failure("Error al subir archivo: $e");
    }
  }

  Future<Result<void>> deleteFile(String path) async {
    try {
      await _storage.ref(path).delete();
      return Success(null);
    } catch (e) {
      return Failure("Error al eliminar: $e");
    }
  }
}
```

---

### Firebase Cloud Messaging (Push Notifications)

```dart
class NotificationService {
  final _messaging = FirebaseMessaging.instance;

  Future<void> initialize() async {
    // Solicitar permisos (iOS requiere esto explícitamente)
    final settings = await _messaging.requestPermission(
      alert: true,
      badge: true,
      sound: true,
      provisional: false,
    );

    if (settings.authorizationStatus == AuthorizationStatus.authorized) {
      await _setupHandlers();
    }
  }

  Future<String?> getToken() => _messaging.getToken();

  Future<void> _setupHandlers() async {
    // App en foreground
    FirebaseMessaging.onMessage.listen(_handleForegroundMessage);

    // App en background — usuario tocó la notificación
    FirebaseMessaging.onMessageOpenedApp.listen(_handleMessageOpenedApp);

    // App terminada — obtener mensaje inicial
    final initialMessage = await _messaging.getInitialMessage();
    if (initialMessage != null) _handleMessageOpenedApp(initialMessage);

    // Suscribirse a topic
    await _messaging.subscribeToTopic('all_users');
  }

  void _handleForegroundMessage(RemoteMessage message) {
    // Mostrar notificación local (ver flutter_local_notifications)
    final notification = message.notification;
    if (notification != null) {
      showLocalNotification(
        title: notification.title ?? '',
        body: notification.body ?? '',
        payload: message.data.toString(),
      );
    }
  }

  void _handleMessageOpenedApp(RemoteMessage message) {
    final route = message.data['route'] as String?;
    if (route != null) navigatorKey.currentContext?.go(route);
  }
}
```

---

## 🗄️ Isar — Base de datos local moderna

```yaml
dependencies:
  isar: ^3.1.7
  isar_flutter_libs: ^3.1.7
  path_provider: ^2.1.5

dev_dependencies:
  isar_generator: ^3.1.7
  build_runner: ^2.4.13
```

```dart
// Esquema
part 'product_schema.g.dart';

@collection
class ProductSchema {
  Id id = Isar.autoIncrement;

  @Index(type: IndexType.value)
  late String remoteId;

  @Index(type: IndexType.value, caseSensitive: false)
  late String name;

  late double price;
  String? category;

  @Index()
  late DateTime createdAt;

  @ignore                        // No persistir este campo
  bool get isExpensive => price > 500;
}

// Inicializar
Future<Isar> openIsar() async {
  final dir = await getApplicationDocumentsDirectory();
  return Isar.open(
    [ProductSchemaSchema],
    directory: dir.path,
    name: 'app_db',
  );
}

// Repositorio con Isar
class ProductIsarRepository {
  ProductIsarRepository(this._isar);
  final Isar _isar;

  // Crear / actualizar (put hace upsert)
  Future<void> save(ProductSchema product) =>
      _isar.writeTxn(() => _isar.productSchemas.put(product));

  Future<void> saveAll(List<ProductSchema> products) =>
      _isar.writeTxn(() => _isar.productSchemas.putAll(products));

  // Leer
  Future<ProductSchema?> getById(String remoteId) =>
      _isar.productSchemas.filter().remoteIdEqualTo(remoteId).findFirst();

  Future<List<ProductSchema>> getAll({String? category}) {
    final query = category != null
        ? _isar.productSchemas.filter().categoryEqualTo(category)
        : _isar.productSchemas.filter().priceGreaterThan(0);
    return query.sortByCreatedAtDesc().findAll();
  }

  // Stream en tiempo real (killer feature de Isar)
  Stream<List<ProductSchema>> watchAll() =>
      _isar.productSchemas.where().sortByCreatedAtDesc().watch(fireImmediately: true);

  Stream<ProductSchema?> watchById(int isarId) =>
      _isar.productSchemas.watchObject(isarId, fireImmediately: true);

  // Eliminar
  Future<bool> delete(int isarId) =>
      _isar.writeTxn(() => _isar.productSchemas.delete(isarId));

  Future<void> deleteAll() =>
      _isar.writeTxn(() => _isar.productSchemas.clear());

  // Query avanzada
  Future<List<ProductSchema>> search(String query) =>
      _isar.productSchemas
          .filter()
          .nameContains(query, caseSensitive: false)
          .or()
          .categoryContains(query, caseSensitive: false)
          .sortByName()
          .limit(50)
          .findAll();

  // Contar
  Future<int> count() => _isar.productSchemas.count();
}
```

---

## 🔒 Seguridad

```yaml
dependencies:
  flutter_secure_storage: ^9.2.2
  local_auth: ^2.3.0
```

```dart
// flutter_secure_storage — tokens y datos sensibles
class SecureStorage {
  static const _storage = FlutterSecureStorage(
    aOptions: AndroidOptions(encryptedSharedPreferences: true),
    iOptions: IOSOptions(accessibility: KeychainAccessibility.first_unlock),
  );

  static const _accessTokenKey = 'access_token';
  static const _refreshTokenKey = 'refresh_token';
  static const _userIdKey = 'user_id';

  Future<void> saveTokens({
    required String accessToken,
    required String refreshToken,
  }) async {
    await Future.wait([
      _storage.write(key: _accessTokenKey, value: accessToken),
      _storage.write(key: _refreshTokenKey, value: refreshToken),
    ]);
  }

  Future<String?> getAccessToken() => _storage.read(key: _accessTokenKey);
  Future<String?> getRefreshToken() => _storage.read(key: _refreshTokenKey);

  Future<void> clearSession() async {
    await Future.wait([
      _storage.delete(key: _accessTokenKey),
      _storage.delete(key: _refreshTokenKey),
      _storage.delete(key: _userIdKey),
    ]);
  }
}

// Autenticación biométrica
class BiometricService {
  final _auth = LocalAuthentication();

  Future<bool> isAvailable() async {
    final canCheck = await _auth.canCheckBiometrics;
    final isDeviceSupported = await _auth.isDeviceSupported();
    return canCheck && isDeviceSupported;
  }

  Future<List<BiometricType>> getAvailableBiometrics() =>
      _auth.getAvailableBiometrics();

  Future<bool> authenticate({
    String reason = 'Confirmá tu identidad para continuar',
  }) async {
    try {
      return await _auth.authenticate(
        localizedReason: reason,
        options: const AuthenticationOptions(
          biometricOnly: false,     // Permitir PIN como fallback
          stickyAuth: true,
        ),
      );
    } on PlatformException {
      return false;
    }
  }

  Future<void> cancelAuthentication() => _auth.stopAuthentication();
}

// Certificate pinning con Dio
class PinningInterceptor extends Interceptor {
  @override
  void onResponse(Response response, ResponseInterceptorHandler handler) {
    // Implementar con dart:io HttpClient para pinning real en producción
    handler.next(response);
  }
}

// Configurar HttpClient con pinning
HttpClient createPinnedClient(List<String> pinnedCertificates) {
  final context = SecurityContext(withTrustedRoots: false);
  for (final cert in pinnedCertificates) {
    context.setTrustedCertificatesBytes(base64Decode(cert));
  }
  return HttpClient(context: context);
}

// Ofuscación — agregar en pubspec.yaml
// flutter build apk --obfuscate --split-debug-info=build/debug-info
// flutter build ipa --obfuscate --split-debug-info=build/debug-info
```

---

## 🏗️ Clean Architecture

```
lib/
├── core/
│   ├── di/                        # Inyección de dependencias (get_it)
│   ├── error/                     # Failures, Exceptions
│   ├── network/                   # DioClient, NetworkInfo
│   ├── storage/                   # SecureStorage, LocalStorage
│   └── utils/                     # Extensions, helpers
├── features/
│   └── products/
│       ├── data/
│       │   ├── datasources/
│       │   │   ├── product_remote_datasource.dart
│       │   │   └── product_local_datasource.dart
│       │   ├── models/            # DTOs con fromJson/toJson
│       │   │   └── product_dto.dart
│       │   └── repositories/      # Implementaciones
│       │       └── product_repository_impl.dart
│       ├── domain/
│       │   ├── entities/          # Modelos de dominio puros (sin Flutter)
│       │   │   └── product.dart
│       │   ├── repositories/      # Interfaces (contratos)
│       │   │   └── product_repository.dart
│       │   └── usecases/          # Un archivo por caso de uso
│       │       ├── get_products.dart
│       │       ├── get_product_by_id.dart
│       │       └── delete_product.dart
│       └── presentation/
│           ├── providers/         # Riverpod providers
│           ├── screens/
│           └── widgets/
```

```dart
// domain/repositories/product_repository.dart
abstract interface class ProductRepository {
  Future<Result<List<Product>>> getAll({String? category});
  Future<Result<Product>> getById(String id);
  Future<Result<String>> create(CreateProductParams params);
  Future<Result<void>> update(String id, UpdateProductParams params);
  Future<Result<void>> delete(String id);
  Stream<List<Product>> watchAll();
}

// domain/usecases/get_products.dart
class GetProductsUseCase {
  const GetProductsUseCase(this._repository);
  final ProductRepository _repository;

  Future<Result<List<Product>>> call({String? category}) =>
      _repository.getAll(category: category);
}

// data/repositories/product_repository_impl.dart
class ProductRepositoryImpl implements ProductRepository {
  const ProductRepositoryImpl({
    required this.remoteDatasource,
    required this.localDatasource,
    required this.networkInfo,
  });

  final ProductRemoteDatasource remoteDatasource;
  final ProductLocalDatasource localDatasource;
  final NetworkInfo networkInfo;

  @override
  Future<Result<List<Product>>> getAll({String? category}) async {
    if (await networkInfo.isConnected) {
      final result = await remoteDatasource.getProducts(category: category);
      switch (result) {
        case Success(:final data):
          await localDatasource.cacheProducts(data);   // Cachear localmente
          return Success(data.map((e) => e.toDomain()).toList());
        case Failure():
          return _getFromCache(category);              // Fallback a cache
      }
    } else {
      return _getFromCache(category);
    }
  }

  Future<Result<List<Product>>> _getFromCache(String? category) async {
    final cached = await localDatasource.getCachedProducts(category: category);
    return cached.isNotEmpty
        ? Success(cached.map((e) => e.toDomain()).toList())
        : Failure("Sin conexión y sin datos en caché");
  }

  @override
  Stream<List<Product>> watchAll() =>
      localDatasource.watchProducts().map(
        (entities) => entities.map((e) => e.toDomain()).toList(),
      );

  @override
  Future<Result<Product>> getById(String id) async {
    final result = await remoteDatasource.getProductById(id);
    return switch (result) {
      Success(:final data) => Success(data.toDomain()),
      Failure(:final message) => Failure(message),
    };
  }

  @override
  Future<Result<String>> create(CreateProductParams params) =>
      remoteDatasource.createProduct(params);

  @override
  Future<Result<void>> update(String id, UpdateProductParams params) =>
      remoteDatasource.updateProduct(id, params);

  @override
  Future<Result<void>> delete(String id) =>
      remoteDatasource.deleteProduct(id);
}

// core/di/injection.dart con get_it
final sl = GetIt.instance;

Future<void> configureDependencies() async {
  // Core
  sl.registerLazySingleton<NetworkInfo>(() => NetworkInfoImpl());
  sl.registerLazySingleton<SecureStorage>(() => SecureStorage());

  final isar = await openIsar();
  sl.registerSingleton<Isar>(isar);

  // Network
  sl.registerLazySingleton(() => DioClient(
    baseUrl: AppConfig.apiBaseUrl,
    tokenStorage: sl(),
    authRepository: sl(),
  ));

  // Datasources
  sl.registerLazySingleton<ProductRemoteDatasource>(
    () => ProductRemoteDatasourceImpl(ProductApiService(sl<DioClient>().instance)),
  );
  sl.registerLazySingleton<ProductLocalDatasource>(
    () => ProductLocalDatasourceImpl(ProductIsarRepository(sl())),
  );

  // Repositories
  sl.registerLazySingleton<ProductRepository>(
    () => ProductRepositoryImpl(
      remoteDatasource: sl(),
      localDatasource: sl(),
      networkInfo: sl(),
    ),
  );

  // Use cases
  sl.registerFactory(() => GetProductsUseCase(sl()));
  sl.registerFactory(() => GetProductByIdUseCase(sl()));
  sl.registerFactory(() => DeleteProductUseCase(sl()));
}
```

---

## 🧭 GoRouter avanzado

```yaml
dependencies:
  go_router: ^14.6.1
```

```dart
// router.dart
final _rootNavigatorKey = GlobalKey<NavigatorState>();
final _shellNavigatorKey = GlobalKey<NavigatorState>();

@riverpod
GoRouter router(RouterRef ref) {
  final authState = ref.watch(authNotifierProvider);

  return GoRouter(
    navigatorKey: _rootNavigatorKey,
    initialLocation: '/splash',
    debugLogDiagnostics: kDebugMode,
    redirect: (context, state) {
      final isAuth = authState.valueOrNull?.isAuthenticated ?? false;
      final isLoading = authState.isLoading;
      final location = state.matchedLocation;

      if (isLoading) return '/splash';

      final publicRoutes = ['/login', '/register', '/forgot-password', '/splash'];
      if (!isAuth && !publicRoutes.contains(location)) return '/login';
      if (isAuth && publicRoutes.contains(location)) return '/home';

      return null;
    },
    routes: [
      GoRoute(path: '/splash', builder: (_, __) => const SplashScreen()),
      GoRoute(path: '/login', builder: (_, __) => const LoginScreen()),
      GoRoute(path: '/register', builder: (_, __) => const RegisterScreen()),
      GoRoute(path: '/forgot-password', builder: (_, __) => const ForgotPasswordScreen()),

      // StatefulShellRoute — bottom nav con estado persistente por tab
      StatefulShellRoute.indexedStack(
        builder: (context, state, navigationShell) =>
            MainShell(navigationShell: navigationShell),
        branches: [
          StatefulShellBranch(
            navigatorKey: _shellNavigatorKey,
            routes: [
              GoRoute(
                path: '/home',
                builder: (_, __) => const HomeScreen(),
                routes: [
                  GoRoute(
                    path: 'product/:id',
                    builder: (_, state) => ProductDetailScreen(
                      id: state.pathParameters['id']!,
                    ),
                  ),
                ],
              ),
            ],
          ),
          StatefulShellBranch(routes: [
            GoRoute(path: '/search', builder: (_, __) => const SearchScreen()),
          ]),
          StatefulShellBranch(routes: [
            GoRoute(
              path: '/profile',
              builder: (_, __) => const ProfileScreen(),
              routes: [
                GoRoute(
                  path: 'settings',
                  builder: (_, __) => const SettingsScreen(),
                ),
              ],
            ),
          ]),
        ],
      ),

      // Modal sobre todas las rutas (navigator raíz)
      GoRoute(
        path: '/cart',
        parentNavigatorKey: _rootNavigatorKey,
        pageBuilder: (context, state) => const MaterialPage(
          fullscreenDialog: true,
          child: CartScreen(),
        ),
      ),
    ],
    errorBuilder: (context, state) => ErrorScreen(error: state.error),
  );
}

// MainShell con BottomNavigationBar
class MainShell extends StatelessWidget {
  const MainShell({super.key, required this.navigationShell});
  final StatefulNavigationShell navigationShell;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: navigationShell,
      bottomNavigationBar: NavigationBar(
        selectedIndex: navigationShell.currentIndex,
        onDestinationSelected: (index) => navigationShell.goBranch(
          index,
          initialLocation: index == navigationShell.currentIndex,
        ),
        destinations: const [
          NavigationDestination(icon: Icon(Icons.home), label: 'Inicio'),
          NavigationDestination(icon: Icon(Icons.search), label: 'Buscar'),
          NavigationDestination(icon: Icon(Icons.person), label: 'Perfil'),
        ],
      ),
    );
  }
}

// Navegación
context.go('/home');                              // Reemplaza stack
context.push('/home/product/123');               // Apila
context.pushReplacement('/login');               // Reemplaza top
context.pop();                                   // Volver
context.canPop();                                // Verificar si puede volver
context.goNamed('product-detail', pathParameters: {'id': '123'});

// Pasar objetos extras (no serializados en URL)
context.push('/detail', extra: product);
// Leer en destino:
final product = state.extra as Product;

// Deep links — configurar en AndroidManifest e Info.plist
// android: android:scheme="https" android:host="miapp.com"
// iOS: CFBundleURLSchemes en Info.plist
```

---

## 🎨 UI avanzada

### CustomPainter

```dart
class CircleProgressPainter extends CustomPainter {
  const CircleProgressPainter({
    required this.progress,
    required this.color,
    this.strokeWidth = 8,
  });

  final double progress;    // 0.0 a 1.0
  final Color color;
  final double strokeWidth;

  @override
  void paint(Canvas canvas, Size size) {
    final center = Offset(size.width / 2, size.height / 2);
    final radius = (size.width - strokeWidth) / 2;

    // Track (fondo)
    final trackPaint = Paint()
      ..color = color.withOpacity(0.2)
      ..style = PaintingStyle.stroke
      ..strokeWidth = strokeWidth;
    canvas.drawCircle(center, radius, trackPaint);

    // Progreso
    final progressPaint = Paint()
      ..color = color
      ..style = PaintingStyle.stroke
      ..strokeWidth = strokeWidth
      ..strokeCap = StrokeCap.round;

    canvas.drawArc(
      Rect.fromCircle(center: center, radius: radius),
      -math.pi / 2,                    // Comenzar desde arriba
      2 * math.pi * progress,          // Ángulo del arco
      false,
      progressPaint,
    );
  }

  @override
  bool shouldRepaint(CircleProgressPainter old) =>
      old.progress != progress || old.color != color;
}

// Uso
CustomPaint(
  size: const Size(100, 100),
  painter: CircleProgressPainter(progress: 0.75, color: Colors.blue),
  child: Center(child: Text("75%")),
)
```

---

### Slivers — scroll personalizado

```dart
CustomScrollView(
  slivers: [
    // AppBar que colapsa al hacer scroll
    SliverAppBar(
      expandedHeight: 250,
      floating: false,
      pinned: true,
      snap: false,
      stretch: true,
      flexibleSpace: FlexibleSpaceBar(
        title: const Text('Productos'),
        background: Image.network(imageUrl, fit: BoxFit.cover),
        stretchModes: const [
          StretchMode.zoomBackground,
          StretchMode.blurBackground,
        ],
      ),
    ),

    // Header fijo
    SliverToBoxAdapter(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: FilterChips(onFilterChanged: onFilterChanged),
      ),
    ),

    // Lista de sección con header sticky
    SliverStickyHeader(
      header: CategoryHeader(title: "Electrónica"),
      sliver: SliverList.builder(
        itemCount: electronics.length,
        itemBuilder: (_, i) => ProductTile(product: electronics[i]),
      ),
    ),

    // Grid
    SliverPadding(
      padding: const EdgeInsets.all(16),
      sliver: SliverGrid.builder(
        gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          childAspectRatio: 0.8,
          crossAxisSpacing: 12,
          mainAxisSpacing: 12,
        ),
        itemCount: products.length,
        itemBuilder: (_, i) => ProductCard(product: products[i]),
      ),
    ),

    // Padding al final para el FAB
    const SliverPadding(padding: EdgeInsets.only(bottom: 80)),
  ],
)
```

---

### LayoutBuilder y MediaQuery — diseño responsive

```dart
// LayoutBuilder — reaccionar al espacio disponible del padre
LayoutBuilder(
  builder: (context, constraints) {
    if (constraints.maxWidth > 900) {
      return const TabletLayout();
    } else if (constraints.maxWidth > 600) {
      return const TabletSmallLayout();
    } else {
      return const MobileLayout();
    }
  },
)

// MediaQuery — información del dispositivo
@Composable
Widget build(BuildContext context) {
  final mq = MediaQuery.of(context);
  final screenWidth = mq.size.width;
  final screenHeight = mq.size.height;
  final topPadding = mq.padding.top;            // Status bar
  final bottomPadding = mq.padding.bottom;      // Home indicator
  final keyboardHeight = mq.viewInsets.bottom;  // Teclado visible
  final isLandscape = mq.orientation == Orientation.landscape;
  final textScaleFactor = mq.textScaler;
  final devicePixelRatio = mq.devicePixelRatio;
  final isDark = mq.platformBrightness == Brightness.dark;

  // Columns responsivos
  final columns = screenWidth > 900 ? 4 : screenWidth > 600 ? 3 : 2;

  return GridView.builder(
    gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
      crossAxisCount: columns,
    ),
    // ...
  );
}

// Breakpoints helper
extension BreakpointExtension on BuildContext {
  bool get isMobile => MediaQuery.of(this).size.width < 600;
  bool get isTablet =>
      MediaQuery.of(this).size.width >= 600 &&
      MediaQuery.of(this).size.width < 900;
  bool get isDesktop => MediaQuery.of(this).size.width >= 900;
}
```

---

### InheritedWidget — compartir datos por el árbol

```dart
// Implementación de InheritedWidget (base de Provider/Riverpod)
class AppData extends InheritedWidget {
  const AppData({
    super.key,
    required this.user,
    required this.theme,
    required super.child,
  });

  final User user;
  final AppTheme theme;

  static AppData of(BuildContext context) {
    final result = context.dependOnInheritedWidgetOfExactType<AppData>();
    assert(result != null, 'No AppData found in context');
    return result!;
  }

  @override
  bool updateShouldNotify(AppData old) =>
      user != old.user || theme != old.theme;
}

// Usar CompositionLocal de Riverpod para esto en la práctica
// InheritedWidget se usa cuando se construyen paquetes/widgets propios
```

---

## 🎬 Animaciones avanzadas

```yaml
dependencies:
  flutter_animate: ^4.5.0
```

### flutter_animate

```dart
// Animaciones declarativas con flutter_animate
Text("Hola")
    .animate()
    .fadeIn(duration: 400.ms)
    .slideX(begin: -0.3, end: 0)

// Encadenar efectos
Container(color: Colors.blue)
    .animate()
    .scale(delay: 200.ms, duration: 400.ms, curve: Curves.elasticOut)
    .then()                               // Secuencial
    .shimmer(duration: 1200.ms)

// Lista animada por índice
ListView.builder(
  itemBuilder: (context, index) =>
      ItemCard(item: items[index])
          .animate(delay: (100 * index).ms)
          .fadeIn()
          .slideY(begin: 0.2),
)

// Animación condicional
Text(isVisible ? "Visible" : "")
    .animate(target: isVisible ? 1.0 : 0.0)
    .fade()
    .scale()

// Loop
Icon(Icons.favorite)
    .animate(onPlay: (controller) => controller.repeat())
    .scale(
      begin: const Offset(1, 1),
      end: const Offset(1.2, 1.2),
      duration: 600.ms,
    )
    .then()
    .scale(end: const Offset(1, 1), duration: 600.ms)
```

---

### AnimationController manual

```dart
class AnimatedLogo extends StatefulWidget {
  const AnimatedLogo({super.key});
  @override
  State<AnimatedLogo> createState() => _AnimatedLogoState();
}

class _AnimatedLogoState extends State<AnimatedLogo>
    with SingleTickerProviderStateMixin {

  late final AnimationController _controller;
  late final Animation<double> _scaleAnimation;
  late final Animation<Color?> _colorAnimation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 800),
    );

    _scaleAnimation = Tween<double>(begin: 0.5, end: 1.0).animate(
      CurvedAnimation(parent: _controller, curve: Curves.elasticOut),
    );

    _colorAnimation = ColorTween(
      begin: Colors.blue,
      end: Colors.purple,
    ).animate(CurvedAnimation(parent: _controller, curve: Curves.easeInOut));

    _controller.forward();
  }

  @override
  void dispose() {
    _controller.dispose();   // SIEMPRE disponer
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: _controller,
      builder: (context, child) => Transform.scale(
        scale: _scaleAnimation.value,
        child: Icon(Icons.star, color: _colorAnimation.value, size: 64),
      ),
    );
  }
}

// Múltiples AnimationControllers
class ComplexAnimation extends StatefulWidget {
  const ComplexAnimation({super.key});
  @override
  State<ComplexAnimation> createState() => _ComplexAnimationState();
}

class _ComplexAnimationState extends State<ComplexAnimation>
    with TickerProviderStateMixin {   // TickerProviderStateMixin para múltiples controllers

  late final AnimationController _fadeController;
  late final AnimationController _slideController;

  @override
  void initState() {
    super.initState();
    _fadeController = AnimationController(vsync: this, duration: 300.ms);
    _slideController = AnimationController(vsync: this, duration: 500.ms);

    // Ejecutar en secuencia
    _fadeController.forward().then((_) => _slideController.forward());
  }

  @override
  void dispose() {
    _fadeController.dispose();
    _slideController.dispose();
    super.dispose();
  }
}

// Transición de ruta personalizada
class SlideUpPageRoute<T> extends PageRouteBuilder<T> {
  SlideUpPageRoute({required this.page})
      : super(
          pageBuilder: (context, animation, secondaryAnimation) => page,
          transitionsBuilder: (context, animation, secondaryAnimation, child) {
            const begin = Offset(0, 1);
            const end = Offset.zero;
            const curve = Curves.easeOutCubic;

            return SlideTransition(
              position: Tween(begin: begin, end: end)
                  .chain(CurveTween(curve: curve))
                  .animate(animation),
              child: child,
            );
          },
          transitionDuration: const Duration(milliseconds: 400),
        );

  final Widget page;
}
```

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
  flutter_local_notifications: ^17.2.4
  timezone: ^0.9.4
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
  permission_handler: ^11.3.1
  image_picker: ^1.1.2
  geolocator: ^13.0.2
  geocoding: ^3.0.0
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
  static Future<bool> requestPhotos() => _request(Permission.photos);
  static Future<bool> requestStorage() => _request(Permission.storage);
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

## ⚡ Isolates y compute

```dart
// compute() — operación pesada sin bloquear UI thread
// Ideal para: parsear JSON grande, encriptar, redimensionar imágenes

// Función top-level o static (requisito de compute)
List<Product> _parseProducts(String jsonString) {
  final list = jsonDecode(jsonString) as List;
  return list.map((e) => Product.fromJson(e as JsonMap)).toList();
}

// Usar en provider/ViewModel
final products = await compute(_parseProducts, response.body);

// Isolate.run (Dart 2.19+) — más simple que Isolate.spawn
Future<Uint8List> processImage(Uint8List imageBytes) async {
  return Isolate.run(() {
    final image = img.decodeImage(imageBytes)!;
    final resized = img.copyResize(image, width: 800);
    return Uint8List.fromList(img.encodeJpg(resized, quality: 85));
  });
}

// Isolate con comunicación bidireccional (para tareas largas)
class HeavyTaskIsolate {
  late Isolate _isolate;
  late SendPort _sendPort;
  final _receivePort = ReceivePort();

  Future<void> start() async {
    _isolate = await Isolate.spawn(_isolateEntry, _receivePort.sendPort);
    _sendPort = await _receivePort.first as SendPort;
  }

  Future<dynamic> runTask(dynamic data) async {
    final responsePort = ReceivePort();
    _sendPort.send([responsePort.sendPort, data]);
    return responsePort.first;
  }

  void stop() {
    _isolate.kill();
    _receivePort.close();
  }

  static void _isolateEntry(SendPort sendPort) {
    final port = ReceivePort();
    sendPort.send(port.sendPort);

    port.listen((message) {
      final responsePort = message[0] as SendPort;
      final data = message[1];
      // Procesar data...
      responsePort.send(result);
    });
  }
}
```

---

## 🌍 Internacionalización (i18n)

```yaml
dependencies:
  flutter_localizations:
    sdk: flutter
  intl: ^0.19.0

flutter:
  generate: true   # Activa la generación automática de código l10n
```

```yaml
# l10n.yaml (en la raíz del proyecto)
arb-dir: lib/l10n
template-arb-file: app_en.arb
output-localization-file: app_localizations.dart
nullable-getter: false
```

```json
// lib/l10n/app_es.arb
{
  "@@locale": "es",
  "appName": "Mi App",
  "welcome": "Bienvenido, {name}",
  "@welcome": {
    "placeholders": {
      "name": { "type": "String" }
    }
  },
  "productCount": "{count, plural, =0{Sin productos} =1{1 producto} other{{count} productos}}",
  "@productCount": {
    "placeholders": {
      "count": { "type": "int" }
    }
  },
  "lastSeen": "Visto {date}",
  "@lastSeen": {
    "placeholders": {
      "date": { "type": "DateTime", "format": "yMMMd" }
    }
  },
  "price": "Precio: {amount}",
  "@price": {
    "placeholders": {
      "amount": { "type": "double", "format": "currency", "optionalParameters": { "symbol": "$" } }
    }
  }
}
```

```dart
// MaterialApp
MaterialApp(
  localizationsDelegates: const [
    AppLocalizations.delegate,
    GlobalMaterialLocalizations.delegate,
    GlobalWidgetsLocalizations.delegate,
    GlobalCupertinoLocalizations.delegate,
  ],
  supportedLocales: AppLocalizations.supportedLocales,
  locale: const Locale('es'),   // Forzar idioma, o null para usar el del sistema
)

// Uso en widgets
final l10n = AppLocalizations.of(context);
Text(l10n.welcome("Juan"))
Text(l10n.productCount(42))
Text(l10n.price(999.99))

// Cambiar idioma en runtime (con Riverpod)
@riverpod
class LocaleNotifier extends _$LocaleNotifier {
  @override
  Locale build() => const Locale('es');

  void setLocale(Locale locale) => state = locale;
}

// MaterialApp con locale dinámica
final locale = ref.watch(localeNotifierProvider);
MaterialApp(locale: locale, ...)

// Formateo de fechas, números y monedas con intl
import 'package:intl/intl.dart';

final dateFormatter = DateFormat.yMMMd('es');
final formatted = dateFormatter.format(DateTime.now());  // "15 ene. 2026"

final currencyFormatter = NumberFormat.currency(locale: 'es_PY', symbol: 'Gs');
final price = currencyFormatter.format(150000);          // "Gs 150.000"

final numberFormatter = NumberFormat.compact(locale: 'es');
final compact = numberFormatter.format(1500000);         // "1,5M"
```

---

## 🧪 Testing completo

```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  mocktail: ^1.0.4
  bloc_test: ^9.1.7
  patrol: ^3.13.0
  golden_toolkit: ^0.15.0
```

```dart
// Unit test con mocktail
class MockProductRepository extends Mock implements ProductRepository {}
class MockApiService extends Mock implements ProductApiService {}

void main() {
  late MockProductRepository mockRepo;
  late GetProductsUseCase useCase;

  setUp(() {
    mockRepo = MockProductRepository();
    useCase = GetProductsUseCase(mockRepo);
  });

  group('GetProductsUseCase', () {
    final tProducts = [
      Product(id: '1', name: 'Laptop', price: 999, imageUrl: ''),
    ];

    test('retorna lista de productos cuando el repositorio tiene éxito', () async {
      when(() => mockRepo.getAll()).thenAnswer((_) async => Success(tProducts));

      final result = await useCase();

      expect(result, isA<Success<List<Product>>>());
      expect((result as Success).data, tProducts);
      verify(() => mockRepo.getAll()).called(1);
    });

    test('retorna Failure cuando el repositorio falla', () async {
      when(() => mockRepo.getAll())
          .thenAnswer((_) async => Failure("Error de red"));

      final result = await useCase();

      expect(result, isA<Failure<List<Product>>>());
      expect((result as Failure).message, "Error de red");
    });
  });
}

// BLoC test
void main() {
  late MockProductRepository mockRepo;

  setUp(() => mockRepo = MockProductRepository());

  blocTest<ProductsCubit, ProductsState>(
    'emite [loading, loaded] cuando loadProducts tiene éxito',
    build: () => ProductsCubit(mockRepo),
    setUp: () => when(() => mockRepo.getAll())
        .thenAnswer((_) async => Success([tProduct])),
    act: (cubit) => cubit.loadProducts(),
    expect: () => [
      const ProductsState.loading(),
      ProductsState.loaded(products: [tProduct]),
    ],
    verify: (_) => verify(() => mockRepo.getAll()).called(1),
  );

  blocTest<ProductsCubit, ProductsState>(
    'emite [loading, error] cuando loadProducts falla',
    build: () => ProductsCubit(mockRepo),
    setUp: () => when(() => mockRepo.getAll())
        .thenAnswer((_) async => Failure("Error")),
    act: (cubit) => cubit.loadProducts(),
    expect: () => [
      const ProductsState.loading(),
      const ProductsState.error(message: "Error"),
    ],
  );
}

// Widget test
void main() {
  testWidgets('ProductCard muestra nombre y precio', (tester) async {
    final product = Product(id: '1', name: 'Laptop', price: 999, imageUrl: '');

    await tester.pumpWidget(
      MaterialApp(
        home: Scaffold(body: ProductCard(product: product)),
      ),
    );

    expect(find.text('Laptop'), findsOneWidget);
    expect(find.text('\$999.00'), findsOneWidget);
    expect(find.byType(ElevatedButton), findsOneWidget);
  });

  testWidgets('ProductCard llama onTap al presionar', (tester) async {
    bool tapped = false;
    await tester.pumpWidget(
      MaterialApp(
        home: ProductCard(product: tProduct, onTap: () => tapped = true),
      ),
    );

    await tester.tap(find.byType(ProductCard));
    await tester.pump();

    expect(tapped, isTrue);
  });

  // Widget test con Riverpod
  testWidgets('ProductsScreen muestra lista', (tester) async {
    await tester.pumpWidget(
      ProviderScope(
        overrides: [
          productsNotifierProvider.overrideWith(() => MockProductsNotifier()),
        ],
        child: const MaterialApp(home: ProductsScreen()),
      ),
    );

    await tester.pumpAndSettle();

    expect(find.byType(ProductCard), findsWidgets);
  });
}

// Golden test — comparar con imagen de referencia
testWidgets('ProductCard golden', (tester) async {
  await tester.pumpWidgetBuilder(
    ProductCard(product: tProduct),
    surfaceSize: const Size(400, 200),
  );

  await expectLater(
    find.byType(ProductCard),
    matchesGoldenFile('goldens/product_card.png'),
  );
});
// Generar imágenes doradas: flutter test --update-goldens
```

---

## ⚡ Performance

```dart
// const constructors — SIEMPRE usar cuando sea posible
// ✅ El widget no se reconstruye si los parámetros no cambian
const Text("Título estático")
const SizedBox(height: 16)
const EdgeInsets.all(16)
const Color(0xFF2563EB)

// RepaintBoundary — aislar partes que se reaniman frecuentemente
RepaintBoundary(
  child: AnimatedProgressBar(value: progress),   // Solo esta parte se repinta
)

// AutomaticKeepAliveClientMixin — mantener estado de tabs en PageView
class ProductsTab extends StatefulWidget { ... }
class _ProductsTabState extends State<ProductsTab>
    with AutomaticKeepAliveClientMixin {

  @override
  bool get wantKeepAlive => true;   // No destruir cuando se cambia de tab

  @override
  Widget build(BuildContext context) {
    super.build(context);           // Llamar super.build SIEMPRE
    return const ProductsList();
  }
}

// Lazy loading de imágenes — Cached Network Image
// implementation: cached_network_image: ^3.4.1
CachedNetworkImage(
  imageUrl: product.imageUrl,
  placeholder: (_, __) => const ShimmerPlaceholder(),
  errorWidget: (_, __, ___) => const Icon(Icons.broken_image),
  memCacheWidth: 300,    // Limitar tamaño en memoria
  memCacheHeight: 300,
)

// Evitar rebuilds innecesarios con select en Riverpod
// ❌ Se reconstruye cuando cualquier campo del usuario cambia
final user = ref.watch(userProvider);
Text(user.name)

// ✅ Solo se reconstruye cuando user.name cambia
final name = ref.watch(userProvider.select((u) => u.name));
Text(name)

// ListView vs ListView.builder
// ❌ Renderiza todos los items de una vez
ListView(children: products.map((p) => ProductCard(p)).toList())

// ✅ Solo renderiza los visibles
ListView.builder(
  itemCount: products.length,
  itemBuilder: (_, i) => ProductCard(products[i]),
)

// Claves para preservar estado en listas dinámicas
ListView.builder(
  itemCount: items.length,
  itemBuilder: (_, i) => ProductCard(
    key: ValueKey(items[i].id),   // Clave única por item
    product: items[i],
  ),
)
```

---

## 🏗️ Flavors y build

```yaml
# pubspec.yaml — dart-define-from-file (Flutter 3.7+)
# Crear archivos de configuración:
# config/dev.json, config/staging.json, config/prod.json
```

```json
// config/dev.json
{
  "FLAVOR": "dev",
  "API_BASE_URL": "https://dev.api.miapp.com/v1",
  "FIREBASE_PROJECT_ID": "miapp-dev",
  "ENABLE_LOGGING": "true",
  "SENTRY_DSN": ""
}

// config/prod.json
{
  "FLAVOR": "prod",
  "API_BASE_URL": "https://api.miapp.com/v1",
  "FIREBASE_PROJECT_ID": "miapp-prod",
  "ENABLE_LOGGING": "false",
  "SENTRY_DSN": "https://key@sentry.io/project"
}
```

```dart
// config/app_config.dart
class AppConfig {
  static const flavor = String.fromEnvironment('FLAVOR', defaultValue: 'dev');
  static const apiBaseUrl = String.fromEnvironment('API_BASE_URL');
  static const enableLogging = bool.fromEnvironment('ENABLE_LOGGING', defaultValue: true);
  static const sentryDsn = String.fromEnvironment('SENTRY_DSN');

  static bool get isDev => flavor == 'dev';
  static bool get isStaging => flavor == 'staging';
  static bool get isProd => flavor == 'prod';
}

// Usar en la app
final dio = Dio(BaseOptions(baseUrl: AppConfig.apiBaseUrl));
if (AppConfig.enableLogging) dio.interceptors.add(PrettyDioLogger());
```

```bash
# Comandos de build con flavors
flutter run --dart-define-from-file=config/dev.json
flutter run --dart-define-from-file=config/staging.json
flutter build apk --dart-define-from-file=config/prod.json --release
flutter build appbundle --dart-define-from-file=config/prod.json --release
flutter build ipa --dart-define-from-file=config/prod.json --release

# Con ofuscación (recomendado para producción)
flutter build apk \
  --dart-define-from-file=config/prod.json \
  --release \
  --obfuscate \
  --split-debug-info=build/debug-info/android

flutter build ipa \
  --dart-define-from-file=config/prod.json \
  --release \
  --obfuscate \
  --split-debug-info=build/debug-info/ios

# Split por ABI (Android — APKs más pequeños)
flutter build apk --split-per-abi --release

# Lanzador de VS Code / Android Studio
# .vscode/launch.json
{
  "configurations": [
    {
      "name": "Dev",
      "request": "launch",
      "type": "dart",
      "args": ["--dart-define-from-file=config/dev.json"]
    },
    {
      "name": "Prod",
      "request": "launch",
      "type": "dart",
      "args": ["--dart-define-from-file=config/prod.json"]
    }
  ]
}
```

---

## 📶 Connectivity

```yaml
dependencies:
  connectivity_plus: ^6.1.1
```

```dart
// NetworkMonitor con Riverpod
@Riverpod(keepAlive: true)
Stream<ConnectivityResult> connectivity(ConnectivityRef ref) =>
    Connectivity().onConnectivityChanged.map((results) => results.first);

@riverpod
bool isOnline(IsOnlineRef ref) {
  final connectivity = ref.watch(connectivityProvider);
  return connectivity.whenData((result) =>
    result != ConnectivityResult.none
  ).value ?? true;
}

// Banner de sin conexión
class ConnectivityBanner extends ConsumerWidget {
  const ConnectivityBanner({super.key, required this.child});
  final Widget child;

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final isOnline = ref.watch(isOnlineProvider);

    return Column(
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
    );
  }
}

// Verificar conectividad antes de una operación
Future<Result<T>> withConnectivity<T>(
  WidgetRef ref,
  Future<Result<T>> Function() operation,
) async {
  if (!ref.read(isOnlineProvider)) {
    return Failure("Sin conexión a internet");
  }
  return operation();
}
```

