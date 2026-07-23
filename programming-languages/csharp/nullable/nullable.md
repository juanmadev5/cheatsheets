# C# — Null Safety (Nullable Reference Types)

---

## 📌 Nullable Reference Types (NRT)

Desde C# 8, el compilador puede advertir sobre posibles `NullReferenceException` en tipos por referencia cuando el contexto `#nullable` está habilitado. En proyectos nuevos (SDK-style, `.NET 6+`) suele venir activado por defecto vía `<Nullable>enable</Nullable>` en el `.csproj`.

### Tabla de resumen

| Sintaxis | Significado | Ejemplo |
|---|---|---|
| **`string`** | No-nullable — el compilador advierte si puede ser `null` | `string name = "Juan";` |
| **`string?`** | Nullable — se permite `null` explícitamente | `string? nickname = null;` |
| **`?.`** | Null-conditional — accede solo si no es `null` | `user?.Name` |
| **`??`** | Operador coalescente — valor por defecto si es `null` | `name ?? "Anónimo"` |
| **`??=`** | Asignación coalescente — asigna solo si es `null` | `cache ??= new()` |
| **`!`** | Null-forgiving — le dice al compilador "confío en que no es null" | `user!.Name` |
| **`?[]`** | Null-conditional en indexadores | `list?[0]` |

---

### Habilitar Nullable Reference Types

```xml
<!-- En el .csproj -->
<PropertyGroup>
  <Nullable>enable</Nullable>
</PropertyGroup>
```

```csharp
// Con Nullable habilitado:
string name = null;    // Warning CS8600: conversión de literal null

#nullable disable
string legacy = null;  // Sin warning — deshabilitado para este bloque/archivo
#nullable restore
```

---

### Operadores de acceso null-safe

```csharp
User? user = FindUser(id);

// ?. — null-conditional, evita NullReferenceException
string? city = user?.Address?.City;
int? nameLength = user?.Name.Length;

// ?[] — null-conditional en indexadores
int? first = numbers?[0];

// ?. encadenado con invocación de método
user?.Notify("Bienvenido");    // No hace nada si user es null

// ?? — valor por defecto
string displayName = user?.Name ?? "Anónimo";

// ??= — asignar solo si es null (C# 8+)
List<string>? cache = null;
cache ??= [];
cache.Add("item");

// ?? con throw
User validUser = user ?? throw new InvalidOperationException("Usuario requerido");

// ! — null-forgiving, silencia el warning del compilador (usar solo si es seguro)
string forced = user!.Name;   // El programador garantiza que user no es null aquí
```

---

### Null-conditional assignment (C# 14)

```csharp
// Antes de C# 14 — había que chequear null antes de asignar
if (customer is not null)
{
    customer.Order = GetCurrentOrder();
}

// C# 14 — ?. del lado izquierdo de una asignación
customer?.Order = GetCurrentOrder();
// El lado derecho solo se evalúa si customer no es null

// También funciona con asignación compuesta
customer?.TotalOrders += 1;

// NO está permitido con ++ / --
// customer?.TotalOrders++;   // Error de compilación
```

---

### Nullable value types — `Nullable<T>`

```csharp
// int? es azúcar sintáctica para Nullable<int>
int? maybeAge = null;
Nullable<int> sameThing = null;

// HasValue / Value
if (maybeAge.HasValue)
{
    Console.WriteLine(maybeAge.Value);
}

// GetValueOrDefault
int age = maybeAge.GetValueOrDefault();       // 0 si es null
int age2 = maybeAge.GetValueOrDefault(18);    // 18 si es null

// Combinable con ?? y ?.
int ageOrDefault = maybeAge ?? 0;

// Conversión explícita a no-nullable
int forced = (int)maybeAge;   // Lanza InvalidOperationException si es null
```

---

### Smart cast / null-state analysis

```csharp
// El compilador rastrea el null-state a través de checks explícitos
string? value = GetValue();
if (value is not null)
{
    Console.WriteLine(value.Length);   // No hay warning — el compilador sabe que no es null acá
}

// Pattern matching con null
if (value is { Length: > 0 } nonEmpty)
{
    Console.WriteLine(nonEmpty);
}

// ArgumentNullException.ThrowIfNull — guard clause idiomática (.NET 6+)
void Process(string? input)
{
    ArgumentNullException.ThrowIfNull(input);
    Console.WriteLine(input.Length);   // input ya se sabe no-null después del throw
}
```

---

## 🛠️ Buenas prácticas

* **Habilitar `<Nullable>enable</Nullable>`** en todo proyecto nuevo — detecta bugs de null en tiempo de compilación.
* **Evitar `!` (null-forgiving)** salvo que realmente esté garantizado — es una promesa sin verificación en runtime.
* **Preferir `??`/`??=`** sobre checks explícitos de `if (x == null)` cuando el objetivo es un valor por defecto.
* **`ArgumentNullException.ThrowIfNull`** en vez de `if (x is null) throw new ArgumentNullException(...)` manual.
