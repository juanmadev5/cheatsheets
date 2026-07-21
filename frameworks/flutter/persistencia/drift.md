# Flutter — Persistencia (SharedPreferences y Drift)

---

## 💾 Persistencia local

```yaml
dependencies:
  shared_preferences: ^2.5.5  # Clave-valor simple
  sqflite: ^2.4.3             # SQLite
  drift: ^2.34.2              # SQL type-safe y reactivo (ver sección Drift)
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

## 🗄️ Drift — Base de datos local moderna

```yaml
dependencies:
  drift: ^2.34.2
  drift_flutter: ^0.3.1   # helper para abrir la DB en Flutter (usa sqlite3 nativo)

dev_dependencies:
  drift_dev: ^2.34.4
  build_runner: ^2.15.2
```

```dart
// database.dart
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';

part 'database.g.dart';

class Products extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get remoteId => text().unique()();
  TextColumn get name => text().withLength(min: 1, max: 200)();
  RealColumn get price => real()();
  TextColumn get category => text().nullable()();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
}

@DriftDatabase(tables: [Products])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());

  @override
  int get schemaVersion => 1;

  static QueryExecutor _openConnection() =>
      driftDatabase(name: 'app_db');   // Elige el backend correcto por plataforma

  // Insert / upsert
  Future<int> upsertProduct(ProductsCompanion product) =>
      into(products).insertOnConflictUpdate(product);

  // Leer todo, ordenado
  Future<List<Product>> getAll({String? category}) {
    final query = select(products)
      ..orderBy([(p) => OrderingTerm.desc(p.createdAt)]);
    if (category != null) {
      query.where((p) => p.category.equals(category));
    }
    return query.get();
  }

  // Stream reactivo — se actualiza solo con cada cambio en la tabla
  Stream<List<Product>> watchAll() =>
      (select(products)..orderBy([(p) => OrderingTerm.desc(p.createdAt)]))
          .watch();

  Future<Product?> getByRemoteId(String remoteId) =>
      (select(products)..where((p) => p.remoteId.equals(remoteId)))
          .getSingleOrNull();

  Future<int> deleteById(int id) =>
      (delete(products)..where((p) => p.id.equals(id))).go();
}

// Uso
final db = AppDatabase();
await db.upsertProduct(ProductsCompanion.insert(
  remoteId: 'abc123',
  name: 'Laptop',
  price: 999.99,
));

// dart run build_runner build --delete-conflicting-outputs
```


