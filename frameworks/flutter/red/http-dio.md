# Flutter — Red (HTTP y Dio)

---

## 🌐 HTTP y APIs

```yaml
dependencies:
  http: ^1.6.0
  dio: ^5.10.0         # Más potente que http
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

## 🌐 Dio — HTTP completo

```yaml
dependencies:
  dio: ^5.10.0
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


