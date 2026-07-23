# C# — Cheatsheet

Cheatsheet de C# (.NET 10 / C# 14) organizado por temas. Cada carpeta contiene el/los `.md` de su categoría.

---

## 📑 Índice

### Base
- [Namespaces/using, tipos de datos y variables, operadores, control de flujo, funciones](base/base.md)

### Null Safety
- [Nullable Reference Types, ?, ?., ??, ??=, !, null-conditional assignment (C# 14)](nullable/nullable.md)

### C# Moderno (9 a 14)
- [Records, primary constructors, pattern matching, collection expressions, field keyword, extension members](csharp-moderno/csharp-moderno.md)

### Programación orientada a objetos
- [Clases, herencia/interfaces, structs, propiedades/indexers, operator overloading](poo/poo.md)

### Programación funcional
- [Delegates, Func/Action, lambdas, extension methods, generics](funcional/funcional.md)

### Colecciones
- [Array, List, Dictionary, HashSet, Queue/Stack, iteradores propios (yield)](colecciones/colecciones.md)

### LINQ
- [Method syntax y query syntax, filtrado, joins, agregación, ejecución diferida](linq/linq.md)

### Async
- [Task, async/await, CancellationToken, IAsyncEnumerable](async/async.md)

### Manejo de errores
- [try/catch/finally, excepciones personalizadas, using/IDisposable](errores/errores.md)

### Standard Library
- [String/StringBuilder, DateTime/DateOnly/TimeOnly, Regex, Math/Convert](stdlib/stdlib.md)

### Herramientas
- [dotnet CLI, .csproj, NuGet, testing con xUnit](herramientas/herramientas.md)

---

## 💡 Buenas prácticas

- **`var`** cuando el tipo es evidente por el lado derecho, tipo explícito cuando mejora la legibilidad.
- **`<Nullable>enable</Nullable>`** en todo proyecto nuevo — detecta bugs de null en tiempo de compilación.
- **`record`** para DTOs y modelos inmutables en vez de clases con boilerplate manual.
- **Primary constructors** para inyección de dependencias simple en servicios y controllers.
- **LINQ method syntax** con `FirstOrDefault`/`SingleOrDefault` en vez de acceso directo cuando el resultado puede no existir.
- **`async`/`await` de punta a punta** — nunca mezclar con `.Result`/`.Wait()`.
- **`throw;` en vez de `throw ex;`** al re-lanzar excepciones — preserva el stack trace.
- **`using`/`await using`** siempre que se trabaje con `IDisposable`/`IAsyncDisposable`.
