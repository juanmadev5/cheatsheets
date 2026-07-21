# Flutter — Performance

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

// Evitar rebuilds innecesarios con context.select en BLoC
// ❌ Se reconstruye cuando cualquier campo del usuario cambia
final user = context.watch<UserCubit>().state;
Text(user.name)

// ✅ Solo se reconstruye cuando user.name cambia
final name = context.select<UserCubit, String>((cubit) => cubit.state.name);
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


