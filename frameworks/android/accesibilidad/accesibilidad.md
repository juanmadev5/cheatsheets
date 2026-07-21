# Android Jetpack Compose — Accesibilidad

---

## ♿ Accesibilidad

```kotlin
// semantics — describir el propósito de un elemento para TalkBack
Box(
    modifier = Modifier
        .semantics {
            contentDescription = "Producto: Laptop, precio: $999"
            role = Role.Button
            onClick(label = "Ver detalles") { true }
        }
        .clickable { }
)

// mergeDescendants — combinar múltiples elementos en uno para TalkBack
Row(modifier = Modifier.semantics(mergeDescendants = true) { }) {
    Icon(Icons.Default.Star, contentDescription = null)  // null → TalkBack lo ignora
    Text("4.5 de 5 estrellas")
}

// CustomAccessibilityAction — acciones adicionales en el menú de TalkBack
Card(
    modifier = Modifier.semantics {
        customActions = listOf(
            CustomAccessibilityAction("Agregar al carrito") {
                onAddToCart()
                true
            },
            CustomAccessibilityAction("Ver detalles") {
                onViewDetails()
                true
            },
        )
    }
) { ProductContent() }

// stateDescription — describir estado actual
Checkbox(
    checked = isChecked,
    onCheckedChange = { isChecked = it },
    modifier = Modifier.semantics {
        stateDescription = if (isChecked) "Seleccionado" else "No seleccionado"
    }
)

// heading — marcar como encabezado
Text(
    "Categorías",
    modifier = Modifier.semantics { heading() },
    style = MaterialTheme.typography.headlineMedium,
)

// LiveRegion — anunciar cambios automáticamente
Text(
    text = statusMessage,
    modifier = Modifier.semantics {
        liveRegion = LiveRegionMode.Polite   // Anuncia cambios sin interrumpir
    }
)

// focusable — hacer enfocable un elemento no interactivo
Box(modifier = Modifier.focusable())

// clearAndSetSemantics — reemplazar completamente la semántica de hijos
Box(
    modifier = Modifier.clearAndSetSemantics {
        contentDescription = "Rating: 4 de 5 estrellas"
    }
) {
    // Cinco íconos de estrella — TalkBack solo leerá el contentDescription de arriba
    repeat(5) { StarIcon() }
}
```

