# Kotlin — Cheatsheet

Cheatsheet de Kotlin organizado por temas. Cada carpeta contiene el/los `.md` de su categoría.

---

## 📑 Índice

### Base
- [Variables/constantes, tipos de datos, null safety, operadores, control de flujo](base/base.md)

### Kotlin moderno (2.0+)
- [when con guards (2.2+), value classes, data classes, sealed classes/interfaces, enum classes](kotlin-moderno/kotlin-moderno.md)

### Programación orientada a objetos
- [Clases, herencia/interfaces, companion objects/objects, delegación, extension functions, operator overloading](poo/poo.md)

### Programación funcional
- [Lambdas, inline/noinline/crossinline, scope functions, funciones de orden superior, generics, type aliases](funcional/funcional.md)

### Colecciones
- [List, Map, Set, Sequence (lazy), builders, destructuring/Pair/Triple](colecciones/colecciones.md)

### Coroutines
- [Fundamentos — launch, async, Dispatchers, withContext, Job/Deferred](coroutines/coroutines.md)
- [Flow, StateFlow/SharedFlow, Channel, exception handling, structured concurrency](coroutines/flow.md)

### Standard Library
- [String, Regex, Result\<T\>, números y conversiones, comparación, kotlin.io/kotlin.system](stdlib/stdlib.md)

### Interoperabilidad con Java
- [@JvmStatic, @JvmOverloads, @JvmField, @JvmName, SAM conversions, platform types](interop/interop.md)

### Build
- [build.gradle.kts completo — plugins, dependencias, testing](build/build.md)

### Referencia
- [Idioms y buenas prácticas, patrones de diseño en Kotlin](referencia/referencia.md)

---

## 💡 Buenas prácticas

- **`data class` en lugar de POJOs** con getters/setters manuales.
- **`val` sobre `var`** siempre que sea posible — inmutabilidad por defecto.
- **Elvis (`?:`) y `let`** para manejo de null en lugar de checks explícitos.
- **`sealed class`/`sealed interface` + `when` exhaustivo** para modelar estados cerrados.
- **Scope functions** (`apply`, `also`, `let`, `run`, `with`) para inicialización y encadenamiento idiomático.
- **`Dispatchers.IO` para I/O, `Dispatchers.Default` para CPU** — nunca bloquear el dispatcher equivocado.
- **`SupervisorJob`** cuando el fallo de una coroutine hija no debe cancelar a sus hermanas.
- **Evitar `GlobalScope`** — usar scopes con lifecycle definido (structured concurrency).
- **`replaceFirstChar { ... }`** en lugar de `capitalize()` (error de compilación desde Kotlin 2.1).
