# Flutter — Clean Architecture

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
│           ├── blocs/             # Cubits / Blocs
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
  sl.registerSingleton<AppDatabase>(AppDatabase());

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
    () => ProductLocalDatasourceImpl(sl<AppDatabase>()),
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


