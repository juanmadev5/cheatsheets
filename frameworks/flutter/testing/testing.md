# Flutter — Testing

---

## 🧪 Testing

```dart
// Unit test
import 'package:flutter_test/flutter_test.dart';

void main() {
  group('CounterCubit', () {
    late CounterCubit counter;

    setUp(() => counter = CounterCubit());

    test('empieza en 0', () {
      expect(counter.state, 0);
    });

    test('incrementa correctamente', () {
      counter.increment();
      expect(counter.state, 1);
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

## 🧪 Testing completo

```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  mocktail: ^1.0.5
  bloc_test: ^10.0.0
  patrol: ^4.7.1
  alchemist: ^0.14.0
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

  // Widget test con BlocProvider mockeado
  // class MockProductsCubit extends MockCubit<ProductsState> implements ProductsCubit {}
  testWidgets('ProductsScreen muestra lista', (tester) async {
    final mockCubit = MockProductsCubit();
    when(() => mockCubit.state)
        .thenReturn(ProductsState.loaded(products: [tProduct]));

    await tester.pumpWidget(
      BlocProvider<ProductsCubit>.value(
        value: mockCubit,
        child: const MaterialApp(home: ProductsScreen()),
      ),
    );

    await tester.pumpAndSettle();

    expect(find.byType(ProductCard), findsWidgets);
  });
}

// Golden test con alchemist — comparar con imagen de referencia
goldenTest(
  'ProductCard luce como se espera',
  fileName: 'product_card',
  builder: () => GoldenTestGroup(
    scenarioConstraints: const BoxConstraints(maxWidth: 400),
    children: [
      GoldenTestScenario(name: 'default', child: ProductCard(product: tProduct)),
      GoldenTestScenario(
        name: 'favorito',
        child: ProductCard(product: tProduct.copyWith(isFavorite: true)),
      ),
    ],
  ),
);
// Generar imágenes doradas: flutter test --update-goldens
```


