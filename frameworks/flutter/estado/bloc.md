# Flutter — Estado (BLoC)

---

## 🧱 BLoC Pattern

```yaml
dependencies:
  flutter_bloc: ^9.1.1
  bloc: ^9.2.1
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


