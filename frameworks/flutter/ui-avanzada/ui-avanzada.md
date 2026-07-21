# Flutter — UI avanzada

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
      ..color = color.withValues(alpha: 0.2)
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
// Implementación de InheritedWidget (mecanismo interno que usan BlocProvider y paquetes similares)
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

// En la práctica, InheritedWidget se usa al construir paquetes/widgets propios;
// para el día a día alcanza con BlocProvider/context.watch
```


