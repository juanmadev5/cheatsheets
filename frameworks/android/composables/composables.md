# Android Jetpack Compose — Composables y UI

---

## 🧱 Composables fundamentales

### Stateless vs Stateful

```kotlin
// Stateless — recibe todo por parámetros (preferido para UI pura)
@Composable
fun UserCard(
    user: User,
    onDelete: () -> Unit,
    modifier: Modifier = Modifier,   // Siempre incluir modifier
) {
    Card(modifier = modifier.fillMaxWidth()) {
        Text(text = user.name)
        IconButton(onClick = onDelete) {
            Icon(Icons.Default.Delete, contentDescription = "Eliminar")
        }
    }
}

// Stateful — gestiona su propio estado (usar lo mínimo posible)
@Composable
fun ExpandableCard(title: String) {
    var expanded by remember { mutableStateOf(false) }

    Card(onClick = { expanded = !expanded }) {
        Text(title)
        AnimatedVisibility(visible = expanded) {
            Text("Contenido expandido...")
        }
    }
}
```

### State hoisting

```kotlin
// Patrón: estado arriba, eventos hacia abajo
@Composable
fun SearchBar(
    query: String,                       // Estado bajando
    onQueryChange: (String) -> Unit,     // Evento subiendo
    modifier: Modifier = Modifier,
) {
    TextField(
        value = query,
        onValueChange = onQueryChange,
        modifier = modifier,
        placeholder = { Text("Buscar...") },
    )
}

// El padre gestiona el estado
@Composable
fun SearchScreen(viewModel: SearchViewModel = koinViewModel()) {
    val query by viewModel.query.collectAsStateWithLifecycle()

    SearchBar(
        query = query,
        onQueryChange = viewModel::onQueryChange,
    )
}
```

---

## 📐 Layout

```kotlin
// Column
Column(
    modifier = Modifier
        .fillMaxSize()
        .padding(16.dp),
    verticalArrangement = Arrangement.spacedBy(8.dp),
    horizontalAlignment = Alignment.CenterHorizontally,
) {
    Text("Primero")
    Text("Segundo")
}

// Row
Row(
    modifier = Modifier.fillMaxWidth(),
    horizontalArrangement = Arrangement.SpaceBetween,
    verticalAlignment = Alignment.CenterVertically,
) {
    Icon(Icons.Default.Home, contentDescription = null)
    Text("Título")
    Icon(Icons.Default.Settings, contentDescription = null)
}

// Box (equivalente a FrameLayout / Stack)
Box(modifier = Modifier.fillMaxSize()) {
    Image(painter = painterResource(R.drawable.bg), contentDescription = null,
        modifier = Modifier.fillMaxSize(), contentScale = ContentScale.Crop)
    Text("Encima", modifier = Modifier.align(Alignment.Center))
    FloatingActionButton(
        onClick = {},
        modifier = Modifier.align(Alignment.BottomEnd).padding(16.dp),
    ) { Icon(Icons.Default.Add, contentDescription = "Agregar") }
}

// LazyColumn — lista eficiente (equivalente a RecyclerView)
LazyColumn(
    contentPadding = PaddingValues(16.dp),
    verticalArrangement = Arrangement.spacedBy(8.dp),
) {
    item { Text("Header") }

    items(items = products, key = { it.id }) { product ->
        ProductCard(product = product)
    }

    itemsIndexed(items = products) { index, product ->
        Text("$index: ${product.name}")
    }

    item { Text("Footer") }
}

// LazyRow — lista horizontal
LazyRow(horizontalArrangement = Arrangement.spacedBy(8.dp)) {
    items(categories) { category ->
        FilterChip(selected = false, onClick = {}, label = { Text(category) })
    }
}
```

### Modifier — operaciones comunes

```kotlin
Modifier
    .fillMaxSize()                              // Ocupa todo el espacio disponible
    .fillMaxWidth()                             // Solo ancho completo
    .fillMaxHeight(0.5f)                        // 50% del alto
    .size(48.dp)                                // Tamaño fijo
    .width(200.dp).height(100.dp)
    .wrapContentSize()                          // Solo el tamaño necesario
    .padding(16.dp)                             // Padding uniforme
    .padding(horizontal = 16.dp, vertical = 8.dp)
    .background(Color.Blue, shape = RoundedCornerShape(8.dp))
    .clip(RoundedCornerShape(12.dp))
    .border(1.dp, Color.Gray, RoundedCornerShape(8.dp))
    .clickable { /* acción */ }
    .clickable(
        interactionSource = remember { MutableInteractionSource() },
        indication = null,                      // Sin ripple
    ) { }
    .semantics { contentDescription = "Botón de compartir" }  // Accesibilidad
    .testTag("share_button")                    // Para tests UI
    .weight(1f)                                 // Dentro de Row/Column
    .align(Alignment.CenterHorizontally)        // Dentro de Column
    .offset(x = 8.dp, y = (-4).dp)
    .rotate(45f)
    .scale(1.2f)
    .alpha(0.5f)
    .zIndex(1f)
```

---

## 🖼️ Widgets visuales comunes

```kotlin
// Texto
Text(
    text = "Hola mundo",
    style = MaterialTheme.typography.headlineMedium,
    color = MaterialTheme.colorScheme.onSurface,
    textAlign = TextAlign.Center,
    maxLines = 2,
    overflow = TextOverflow.Ellipsis,
    fontWeight = FontWeight.Bold,
)

// Texto con múltiples estilos
Text(
    text = buildAnnotatedString {
        append("Precio: ")
        withStyle(SpanStyle(fontWeight = FontWeight.Bold, color = Color.Green)) {
            append("\$99.99")
        }
    }
)

// Imagen con Coil
AsyncImage(
    model = ImageRequest.Builder(LocalContext.current)
        .data(product.imageUrl)
        .crossfade(true)
        .build(),
    contentDescription = product.name,
    modifier = Modifier.fillMaxWidth().height(200.dp),
    contentScale = ContentScale.Crop,
    placeholder = painterResource(R.drawable.placeholder),
    error = painterResource(R.drawable.error),
)

// Botones
Button(onClick = {}) { Text("Primario") }
OutlinedButton(onClick = {}) { Text("Outlined") }
TextButton(onClick = {}) { Text("Text") }
FilledTonalButton(onClick = {}) { Text("Tonal") }
ElevatedButton(onClick = {}) { Text("Elevated") }
IconButton(onClick = {}) { Icon(Icons.Default.Favorite, contentDescription = "Like") }
FloatingActionButton(onClick = {}) { Icon(Icons.Default.Add, contentDescription = "Add") }
ExtendedFloatingActionButton(
    onClick = {},
    icon = { Icon(Icons.Default.Add, contentDescription = null) },
    text = { Text("Nuevo") },
)

// TextField
var text by remember { mutableStateOf("") }
OutlinedTextField(
    value = text,
    onValueChange = { text = it },
    label = { Text("Email") },
    leadingIcon = { Icon(Icons.Default.Email, contentDescription = null) },
    trailingIcon = { if (text.isNotEmpty()) IconButton(onClick = { text = "" }) {
        Icon(Icons.Default.Clear, contentDescription = "Limpiar")
    }},
    keyboardOptions = KeyboardOptions(
        keyboardType = KeyboardType.Email,
        imeAction = ImeAction.Next,
    ),
    keyboardActions = KeyboardActions(onNext = { /* focus next */ }),
    singleLine = true,
    isError = text.isNotEmpty() && !text.contains("@"),
    supportingText = { if (text.isNotEmpty() && !text.contains("@")) Text("Email inválido") },
    modifier = Modifier.fillMaxWidth(),
)
```

