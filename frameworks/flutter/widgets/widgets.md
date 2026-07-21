# Flutter — Widgets

---

## 🧱 Widgets fundamentales

### StatelessWidget vs StatefulWidget

```dart
// StatelessWidget — sin estado interno
class MyWidget extends StatelessWidget {
  final String title;
  const MyWidget({super.key, required this.title});

  @override
  Widget build(BuildContext context) {
    return Text(title);
  }
}

// StatefulWidget — con estado interno mutable
class Counter extends StatefulWidget {
  const Counter({super.key});

  @override
  State<Counter> createState() => _CounterState();
}

class _CounterState extends State<Counter> {
  int _count = 0;

  @override
  Widget build(BuildContext context) {
    return TextButton(
      onPressed: () => setState(() => _count++),
      child: Text('Count: $_count'),
    );
  }
}
```



---

## 📐 Layout — Widgets de estructura

### Column y Row

```dart
Column(
  mainAxisAlignment: MainAxisAlignment.center,    // Eje principal (vertical)
  crossAxisAlignment: CrossAxisAlignment.start,   // Eje cruzado (horizontal)
  mainAxisSize: MainAxisSize.min,                 // Tamaño mínimo necesario
  children: [
    Text('Primero'),
    const SizedBox(height: 8),
    Text('Segundo'),
  ],
)

Row(
  mainAxisAlignment: MainAxisAlignment.spaceBetween,
  children: [
    Icon(Icons.home),
    Text('Título'),
    Icon(Icons.settings),
  ],
)
```

### Stack

```dart
Stack(
  alignment: Alignment.center,
  children: [
    Image.network('https://...'),
    Positioned(
      bottom: 16,
      right: 16,
      child: FloatingActionButton(onPressed: () {}, child: const Icon(Icons.add)),
    ),
  ],
)
```

### Expanded y Flexible

```dart
Row(
  children: [
    Expanded(child: Container(color: Colors.red)),     // Ocupa todo el espacio disponible
    Expanded(flex: 2, child: Container(color: Colors.blue)), // El doble de espacio
    Flexible(child: Text('Solo lo necesario')),        // Solo el espacio que necesita
  ],
)
```

### Container

```dart
Container(
  width: 200,
  height: 100,
  margin: const EdgeInsets.all(16),
  padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 8),
  decoration: BoxDecoration(
    color: Colors.blue.shade100,
    borderRadius: BorderRadius.circular(12),
    border: Border.all(color: Colors.blue, width: 1.5),
    boxShadow: [
      BoxShadow(color: Colors.black26, blurRadius: 8, offset: Offset(0, 4)),
    ],
  ),
  child: const Text('Hola'),
)
```

### Otros widgets de layout

| Widget | Descripción |
|---|---|
| `SizedBox(width, height)` | Espacio fijo o contenedor de tamaño exacto |
| `Padding(padding:, child:)` | Agregar padding |
| `Center(child:)` | Centrar su hijo |
| `Align(alignment:, child:)` | Alinear hijo en posición específica |
| `AspectRatio(aspectRatio:)` | Mantener proporción |
| `FractionallySizedBox` | Tamaño como fracción del padre |
| `ConstrainedBox` | Aplicar constraints mínimos/máximos |
| `Wrap` | Como Row/Column pero con wrap |
| `Spacer()` | Espacio flexible entre widgets |
| `IntrinsicHeight` / `IntrinsicWidth` | Igualar alturas/anchos de hijos |



---

## 📋 Widgets de listas

```dart
// Lista simple
ListView(
  padding: const EdgeInsets.all(16),
  children: [Text('Item 1'), Text('Item 2')],
)

// Lista eficiente (lazy) — SIEMPRE usar para listas largas
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) => ListTile(
    leading: const Icon(Icons.person),
    title: Text(items[index].name),
    subtitle: Text(items[index].email),
    trailing: const Icon(Icons.chevron_right),
    onTap: () {},
  ),
)

// Lista separada
ListView.separated(
  itemCount: items.length,
  separatorBuilder: (_, __) => const Divider(),
  itemBuilder: (context, index) => Text(items[index]),
)

// Grid
GridView.builder(
  gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
    crossAxisCount: 2,
    childAspectRatio: 1.5,
    crossAxisSpacing: 8,
    mainAxisSpacing: 8,
  ),
  itemCount: items.length,
  itemBuilder: (context, index) => Card(child: Text(items[index])),
)
```



---

## 🎨 Widgets visuales comunes

### Text

```dart
Text(
  'Hola mundo',
  style: TextStyle(
    fontSize: 24,
    fontWeight: FontWeight.bold,
    color: Colors.blue,
    letterSpacing: 1.2,
    height: 1.5,            // Line height
    decoration: TextDecoration.underline,
  ),
  textAlign: TextAlign.center,
  maxLines: 2,
  overflow: TextOverflow.ellipsis,
)

// Rich text con múltiples estilos
RichText(
  text: TextSpan(
    text: 'Hola ',
    style: DefaultTextStyle.of(context).style,
    children: [
      TextSpan(text: 'mundo', style: TextStyle(fontWeight: FontWeight.bold)),
      TextSpan(text: '!', style: TextStyle(color: Colors.red)),
    ],
  ),
)
```

### Imagen

```dart
Image.asset('assets/images/logo.png', width: 100, fit: BoxFit.contain)
Image.network('https://...', fit: BoxFit.cover, errorBuilder: (ctx, err, st) => Icon(Icons.error))

// Imagen circular
CircleAvatar(
  radius: 30,
  backgroundImage: NetworkImage('https://...'),
  child: null,   // Si se usa backgroundImage, child queda encima
)
```

### Botones

```dart
ElevatedButton(onPressed: () {}, child: const Text('Elevado'))
OutlinedButton(onPressed: () {}, child: const Text('Outlined'))
TextButton(onPressed: () {}, child: const Text('Text'))
IconButton(onPressed: () {}, icon: const Icon(Icons.favorite))
FilledButton(onPressed: () {}, child: const Text('Filled'))  // Material 3

// Con ícono
ElevatedButton.icon(
  onPressed: () {},
  icon: const Icon(Icons.send),
  label: const Text('Enviar'),
)
```

### Input / Forms

```dart
// Campo de texto
TextField(
  controller: _controller,
  decoration: const InputDecoration(
    labelText: 'Email',
    hintText: 'tu@email.com',
    prefixIcon: Icon(Icons.email),
    border: OutlineInputBorder(),
    errorText: 'Campo requerido',   // null = sin error
  ),
  keyboardType: TextInputType.emailAddress,
  obscureText: false,
  maxLines: 1,
  onChanged: (value) {},
  onSubmitted: (value) {},
)

// Form con validación
final _formKey = GlobalKey<FormState>();

Form(
  key: _formKey,
  child: Column(
    children: [
      TextFormField(
        validator: (value) {
          if (value == null || value.isEmpty) return 'Campo requerido';
          return null;  // null = válido
        },
        decoration: const InputDecoration(labelText: 'Nombre'),
      ),
      ElevatedButton(
        onPressed: () {
          if (_formKey.currentState!.validate()) {
            // Formulario válido
          }
        },
        child: const Text('Enviar'),
      ),
    ],
  ),
)
```



---

## 🧭 Scaffold y navegación básica

```dart
Scaffold(
  appBar: AppBar(
    title: const Text('Mi Página'),
    actions: [IconButton(icon: const Icon(Icons.search), onPressed: () {})],
    leading: const BackButton(),
  ),
  body: const Center(child: Text('Contenido')),
  floatingActionButton: FloatingActionButton(
    onPressed: () {},
    child: const Icon(Icons.add),
  ),
  bottomNavigationBar: BottomNavigationBar(
    currentIndex: _index,
    onTap: (i) => setState(() => _index = i),
    items: const [
      BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Inicio'),
      BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Perfil'),
    ],
  ),
  drawer: Drawer(child: ListView(/* ... */)),
)
```

### Navigator (navegación imperativa)

```dart
// Ir a otra pantalla
Navigator.push(context, MaterialPageRoute(builder: (_) => const DetailPage()));

// Ir y reemplazar pantalla actual
Navigator.pushReplacement(context, MaterialPageRoute(builder: (_) => const HomePage()));

// Ir y eliminar todas las rutas anteriores
Navigator.pushAndRemoveUntil(
  context,
  MaterialPageRoute(builder: (_) => const HomePage()),
  (route) => false,
);

// Volver
Navigator.pop(context);

// Volver con resultado
Navigator.pop(context, 'resultado');

// Esperar resultado
final result = await Navigator.push(context, MaterialPageRoute(builder: (_) => const Picker()));
```



---

## 🎭 Ciclo de vida de widgets

```dart
class MyPage extends StatefulWidget {
  const MyPage({super.key});

  @override
  State<MyPage> createState() => _MyPageState();
}

class _MyPageState extends State<MyPage> {
  late TextEditingController _controller;
  late ScrollController _scrollController;

  @override
  void initState() {
    super.initState();                    // SIEMPRE llamar primero
    _controller = TextEditingController();
    _scrollController = ScrollController();
    _loadData();                          // Inicializar datos async
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();        // Llamado cuando dependencias cambian
    // Seguro acceder a context.watch() aquí
  }

  @override
  void didUpdateWidget(MyPage oldWidget) {
    super.didUpdateWidget(oldWidget);     // Widget padre reconstruyó con nuevos props
  }

  @override
  void dispose() {
    _controller.dispose();               // SIEMPRE liberar controllers
    _scrollController.dispose();
    super.dispose();                      // SIEMPRE llamar al final
  }

  Future<void> _loadData() async { /* ... */ }

  @override
  Widget build(BuildContext context) => Container();
}
```


