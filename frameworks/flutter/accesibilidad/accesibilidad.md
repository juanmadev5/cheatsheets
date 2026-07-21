# Flutter — Accesibilidad

---

## ♿ Semantics — describir la UI para TalkBack/VoiceOver

```dart
// Semantics envuelve cualquier widget y describe su propósito para el lector de pantalla
Semantics(
  label: 'Agregar producto al carrito',
  button: true,
  onTap: () => addToCart(product),
  child: IconButton(
    icon: const Icon(Icons.add_shopping_cart),
    onPressed: () => addToCart(product),
  ),
)

// Imágenes decorativas — excluir del árbol de accesibilidad
ExcludeSemantics(
  child: Image.asset('assets/decoracion.png'),
)

// Imagen con significado — describirla explícitamente
Image.asset(
  'assets/logo.png',
  semanticLabel: 'Logo de la empresa',
)

// mergeSemantics — combinar varios widgets en un solo nodo (una card entera como un solo elemento)
MergeSemantics(
  child: Row(
    children: [
      const Icon(Icons.star, semanticLabel: null),
      Text('4.5 de 5 estrellas'),
    ],
  ),
)

// stateDescription — describir un estado (ej: switches, checkboxes custom)
Semantics(
  label: 'Notificaciones',
  toggled: isEnabled,
  child: Switch(value: isEnabled, onChanged: onToggle),
)

// liveRegion — anunciar cambios automáticamente sin que el usuario tenga que enfocar el widget
Semantics(
  liveRegion: true,
  child: Text(statusMessage),   // Ej: "3 productos agregados al carrito"
)

// Encabezados — ayuda a navegar por secciones con gestos de TalkBack/VoiceOver
Semantics(
  header: true,
  child: Text('Categorías', style: Theme.of(context).textTheme.headlineSmall),
)

// sortKey — controlar el orden de lectura cuando difiere del orden visual
Semantics(
  sortKey: const OrdinalSortKey(1),
  child: const Text('Se lee primero'),
)
```

---

## 🎯 Tamaño de touch targets

```dart
// Android recomienda mínimo 48x48dp, iOS 44x44pt — usar constraints para garantizarlo
IconButton(
  constraints: const BoxConstraints(minWidth: 48, minHeight: 48),
  icon: const Icon(Icons.close),
  onPressed: onClose,
)

// Widgets sin feedback táctil nativo (ej: un Container clickable) — envolver en InkWell/GestureDetector
// y asegurar el área mínima con un padding o SizedBox
```

---

## 🌓 Contraste y escalado de texto

```dart
// Respetar el textScaleFactor del sistema — nunca hardcodear tamaños de fuente sin MediaQuery
final textScaler = MediaQuery.textScalerOf(context);

// Evitar textos con contraste insuficiente — usar los colorScheme.on* de Material 3,
// que ya están diseñados para cumplir WCAG AA sobre su color de fondo correspondiente
Text('Precio', style: TextStyle(color: Theme.of(context).colorScheme.onSurface))

// Permitir que el layout crezca con el texto en vez de truncarlo silenciosamente
Text(
  producto.nombre,
  maxLines: 2,
  overflow: TextOverflow.ellipsis,
)
```

---

## 🧪 Testing de accesibilidad

```dart
testWidgets('el botón cumple las guías de accesibilidad', (tester) async {
  final handle = tester.ensureSemantics();

  await tester.pumpWidget(const MyApp());

  // Tamaño mínimo de touch target según plataforma
  await expectLater(tester, meetsGuideline(androidTapTargetGuideline));
  await expectLater(tester, meetsGuideline(iOSTapTargetGuideline));

  // Todo elemento tocable tiene un label
  await expectLater(tester, meetsGuideline(labeledTapTargetGuideline));

  // Contraste mínimo de texto (WCAG)
  await expectLater(tester, meetsGuideline(textContrastGuideline));

  handle.dispose();
});

// Verificar el árbol de semántica generado por un widget puntual
testWidgets('el botón expone el label correcto', (tester) async {
  await tester.pumpWidget(const MyApp());

  expect(
    tester.getSemantics(find.byIcon(Icons.add_shopping_cart)),
    matchesSemantics(label: 'Agregar producto al carrito', isButton: true),
  );
});
```

> En dispositivo/emulador: activar TalkBack (Android, `Ajustes > Accesibilidad`) o VoiceOver (iOS, `Ajustes > Accesibilidad`) y navegar la app completa con gestos, sin usar la vista.
