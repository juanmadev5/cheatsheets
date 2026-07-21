# Android Jetpack Compose — UI avanzada

---

## 🎛️ Compose — widgets avanzados

### PullToRefresh

```kotlin
@Composable
fun ProductsScreen(viewModel: ProductsViewModel = koinViewModel()) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    val isRefreshing = uiState is UiState.Loading

    val pullToRefreshState = rememberPullToRefreshState()

    PullToRefreshBox(
        isRefreshing = isRefreshing,
        onRefresh = viewModel::refresh,
        state = pullToRefreshState,
    ) {
        LazyColumn(modifier = Modifier.fillMaxSize()) {
            // contenido
        }
    }
}
```

---

### SwipeToDismiss

```kotlin
@Composable
fun SwipeableProductCard(
    product: Product,
    onDelete: (Product) -> Unit,
) {
    val dismissState = rememberSwipeToDismissBoxState(
        confirmValueChange = { value ->
            if (value == SwipeToDismissBoxValue.EndToStart) {
                onDelete(product)
                true
            } else false
        }
    )

    SwipeToDismissBox(
        state = dismissState,
        backgroundContent = {
            val color by animateColorAsState(
                targetValue = if (dismissState.dismissDirection == SwipeToDismissBoxValue.EndToStart)
                    MaterialTheme.colorScheme.errorContainer else Color.Transparent,
                label = "swipe_bg",
            )
            Box(
                modifier = Modifier.fillMaxSize().background(color).padding(end = 16.dp),
                contentAlignment = Alignment.CenterEnd,
            ) {
                Icon(Icons.Default.Delete, contentDescription = "Eliminar",
                    tint = MaterialTheme.colorScheme.onErrorContainer)
            }
        },
        enableDismissFromStartToEnd = false,
    ) {
        ProductCard(product = product)
    }
}
```

---

### ModalBottomSheet, DatePicker y SearchBar

```kotlin
// ModalBottomSheet
var showSheet by remember { mutableStateOf(false) }

if (showSheet) {
    ModalBottomSheet(
        onDismissRequest = { showSheet = false },
        sheetState = rememberModalBottomSheetState(skipPartiallyExpanded = true),
    ) {
        Column(modifier = Modifier.padding(16.dp).navigationBarsPadding()) {
            Text("Opciones", style = MaterialTheme.typography.titleLarge)
            Spacer(Modifier.height(16.dp))
            // contenido del sheet
        }
    }
}

// DatePicker (Material 3)
var showDatePicker by remember { mutableStateOf(false) }
val datePickerState = rememberDatePickerState(
    initialSelectedDateMillis = System.currentTimeMillis()
)

if (showDatePicker) {
    DatePickerDialog(
        onDismissRequest = { showDatePicker = false },
        confirmButton = {
            TextButton(onClick = {
                showDatePicker = false
                val selectedMillis = datePickerState.selectedDateMillis
                // usar selectedMillis
            }) { Text("Confirmar") }
        },
        dismissButton = {
            TextButton(onClick = { showDatePicker = false }) { Text("Cancelar") }
        },
    ) {
        DatePicker(state = datePickerState)
    }
}

// TimePicker (Material 3)
val timePickerState = rememberTimePickerState(initialHour = 12, initialMinute = 0)
TimePicker(state = timePickerState)
// Leer: timePickerState.hour, timePickerState.minute

// SearchBar (Material 3)
var query by remember { mutableStateOf("") }
var active by remember { mutableStateOf(false) }

SearchBar(
    inputField = {
        SearchBarDefaults.InputField(
            query = query,
            onQueryChange = { query = it },
            onSearch = { active = false; viewModel.search(query) },
            expanded = active,
            onExpandedChange = { active = it },
            placeholder = { Text("Buscar productos...") },
            leadingIcon = { Icon(Icons.Default.Search, contentDescription = null) },
            trailingIcon = {
                if (query.isNotEmpty()) {
                    IconButton(onClick = { query = "" }) {
                        Icon(Icons.Default.Clear, contentDescription = "Limpiar")
                    }
                }
            },
        )
    },
    expanded = active,
    onExpandedChange = { active = it },
    modifier = Modifier.fillMaxWidth(),
) {
    // Sugerencias de búsqueda cuando está expandido
    LazyColumn {
        items(suggestions) { suggestion ->
            ListItem(
                headlineContent = { Text(suggestion) },
                modifier = Modifier.clickable { query = suggestion; active = false }
            )
        }
    }
}

// ExposedDropdownMenu
var expanded by remember { mutableStateOf(false) }
var selectedOption by remember { mutableStateOf(options.first()) }

ExposedDropdownMenuBox(
    expanded = expanded,
    onExpandedChange = { expanded = it },
) {
    OutlinedTextField(
        value = selectedOption,
        onValueChange = {},
        readOnly = true,
        label = { Text("Categoría") },
        trailingIcon = { ExposedDropdownMenuDefaults.TrailingIcon(expanded) },
        modifier = Modifier.menuAnchor().fillMaxWidth(),
    )
    ExposedDropdownMenu(
        expanded = expanded,
        onDismissRequest = { expanded = false },
    ) {
        options.forEach { option ->
            DropdownMenuItem(
                text = { Text(option) },
                onClick = { selectedOption = option; expanded = false },
            )
        }
    }
}

// LazyColumn con stickyHeader
LazyColumn {
    groupedItems.forEach { (header, items) ->
        stickyHeader {
            Surface(
                modifier = Modifier.fillMaxWidth(),
                color = MaterialTheme.colorScheme.surfaceVariant,
            ) {
                Text(
                    text = header,
                    modifier = Modifier.padding(horizontal = 16.dp, vertical = 8.dp),
                    style = MaterialTheme.typography.labelLarge,
                )
            }
        }
        items(items = items, key = { it.id }) { item ->
            ItemCard(item = item)
        }
    }
}
```

---

## 🎬 Animaciones

```kotlin
// AnimatedVisibility
AnimatedVisibility(
    visible = isVisible,
    enter = fadeIn() + slideInVertically(),
    exit = fadeOut() + slideOutVertically(),
) {
    Text("Aparezco animado")
}

// animateContentSize — anima cambios de tamaño
Box(modifier = Modifier.animateContentSize(animationSpec = spring())) {
    Text(if (expanded) longText else shortText)
}

// animate*AsState — animar un valor
val alpha by animateFloatAsState(
    targetValue = if (isVisible) 1f else 0f,
    animationSpec = tween(300),
    label = "alpha",
)
val color by animateColorAsState(
    targetValue = if (isSelected) MaterialTheme.colorScheme.primary else Color.Gray,
    label = "color",
)

// Crossfade entre composables
Crossfade(targetState = currentScreen, label = "screen_transition") { screen ->
    when (screen) {
        Screen.Home -> HomeContent()
        Screen.Detail -> DetailContent()
    }
}

// updateTransition — animaciones coordinadas
val transition = updateTransition(targetState = isExpanded, label = "expand")
val height by transition.animateDp(label = "height") { expanded -> if (expanded) 200.dp else 56.dp }
val alpha by transition.animateFloat(label = "alpha") { expanded -> if (expanded) 1f else 0f }

// Animación con rememberInfiniteTransition
val infiniteTransition = rememberInfiniteTransition(label = "pulse")
val scale by infiniteTransition.animateFloat(
    initialValue = 1f,
    targetValue = 1.1f,
    animationSpec = infiniteRepeatable(
        animation = tween(1000),
        repeatMode = RepeatMode.Reverse,
    ),
    label = "scale",
)
```

