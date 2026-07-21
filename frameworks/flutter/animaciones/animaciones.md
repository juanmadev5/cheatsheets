# Flutter — Animaciones

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

## 🎬 Animaciones avanzadas

```yaml
dependencies:
  flutter_animate: ^4.5.2
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


