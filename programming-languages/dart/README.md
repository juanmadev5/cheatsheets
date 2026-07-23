# Dart — Cheatsheet

Cheatsheet de Dart organizado por temas. Cada carpeta contiene el/los `.md` de su categoría.

---

## 📑 Índice

### Base
- [Tipos de datos y variables, operadores, control de flujo, funciones](base/base.md)

### Null Safety
- [?, ?., ??, ??=, !, late, required](null-safety/null-safety.md)

### Dart 3 moderno
- [Records y destructuring, pattern matching, dot shorthands, sealed classes y class modifiers, extension types](dart-moderno/dart-moderno.md)

### Programación orientada a objetos
- [Clases, herencia/interfaces, mixins, extensions, enums avanzados, generics, typedefs](poo/poo.md)

### Colecciones
- [Listas, mapas, sets, operaciones funcionales, spread/collection if/for](colecciones/colecciones.md)

### Async
- [Future, Stream, Isolates](async/async.md)

### Manejo de errores
- [Excepciones — throw, try/catch/on/finally, custom exceptions, assert](errores/errores.md)

### Sistema de tipos avanzado
- [Type checks, promotion, callable classes, funciones de primera clase y closures](tipos-avanzados/tipos-avanzados.md)

### Dart Core
- [String, DateTime/Duration, dart:math, RegExp](core/core.md)

### Entrada/Salida
- [dart:io — archivos y directorios, dart:convert — JSON y codificación](io/io.md)

### Herramientas
- [dart CLI, pubspec.yaml, buenas prácticas y patrones de diseño](herramientas/herramientas.md)

---

## 💡 Buenas prácticas

- **Preferir `final` sobre `var`** cuando el valor no se reasigna.
- **Usar `const`** para valores conocidos en tiempo de compilación — reduce allocations.
- **`?.` y `??`** en lugar de checks explícitos de null.
- **Records para retornar múltiples valores** en lugar de crear clases wrapper o usar `Map`.
- **`sealed class` + `switch` exhaustivo** para modelar estados cerrados (evita el `else` con lógica oculta).
- **Extensions en lugar de utility classes** para agregar comportamiento a tipos existentes.
- **`Isolate.run()`** para trabajo pesado — no bloquear el isolate principal.
- **`dart format` y `dart analyze`** regularmente para mantener el código consistente.
