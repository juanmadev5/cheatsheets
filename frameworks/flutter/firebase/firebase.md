# Flutter — Firebase

---

## 🔥 Firebase completo

```yaml
dependencies:
  firebase_core: ^4.12.1
  firebase_auth: ^6.5.6
  cloud_firestore: ^6.7.1
  firebase_storage: ^13.4.5
  firebase_messaging: ^16.4.3
  firebase_crashlytics: ^5.2.6
  firebase_analytics: ^12.4.5
  google_sign_in: ^7.2.0
  sign_in_with_apple: ^8.1.0
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

  runApp(const MyApp());
}
```

---

### Firebase Auth

```dart
class AuthRepository {
  final _auth = FirebaseAuth.instance;
  final _googleSignIn = GoogleSignIn.instance;

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

  // Google Sign-In — inicializar una sola vez al arrancar la app:
  // await GoogleSignIn.instance.initialize(serverClientId: '...apps.googleusercontent.com');
  Future<Result<User>> signInWithGoogle() async {
    try {
      final googleUser = await _googleSignIn.authenticate();
      final idToken = googleUser.authentication.idToken;
      final credential = GoogleAuthProvider.credential(idToken: idToken);
      final cred = await _auth.signInWithCredential(credential);
      return Success(cred.user!);
    } on GoogleSignInException {
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
    await _googleSignIn.signOut();
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


