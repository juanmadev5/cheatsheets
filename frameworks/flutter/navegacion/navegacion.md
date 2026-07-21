# Flutter — Navegación (GoRouter)

---

## 🗺️ GoRouter (navegación declarativa)

```yaml
# pubspec.yaml
dependencies:
  go_router: ^17.3.0
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

## 🧭 GoRouter avanzado

```yaml
dependencies:
  go_router: ^17.3.0
```

```dart
// Puente entre un Stream (el .stream de un Bloc/Cubit) y Listenable,
// para que GoRouter recalcule el redirect cuando cambia el AuthBloc
class GoRouterRefreshStream extends ChangeNotifier {
  GoRouterRefreshStream(Stream<dynamic> stream) {
    notifyListeners();
    _subscription = stream.listen((_) => notifyListeners());
  }

  late final StreamSubscription<dynamic> _subscription;

  @override
  void dispose() {
    _subscription.cancel();
    super.dispose();
  }
}

// router.dart
final _rootNavigatorKey = GlobalKey<NavigatorState>();
final _shellNavigatorKey = GlobalKey<NavigatorState>();

GoRouter buildRouter(AuthBloc authBloc) {
  return GoRouter(
    navigatorKey: _rootNavigatorKey,
    initialLocation: '/splash',
    debugLogDiagnostics: kDebugMode,
    refreshListenable: GoRouterRefreshStream(authBloc.stream),
    redirect: (context, state) {
      final authState = authBloc.state;
      final isAuth = authState is AuthAuthenticated;
      final isLoading = authState is AuthLoading;
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

// main.dart — construir el router una vez con el AuthBloc del service locator
final router = buildRouter(sl<AuthBloc>());
runApp(MaterialApp.router(routerConfig: router));

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


