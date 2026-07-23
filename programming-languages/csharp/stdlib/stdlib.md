# C# — Standard Library

---

## 📌 String

```csharp
string s = "Hola Mundo";

// Interpolación y formato
string interpolated = $"Nombre: {name}, Edad: {age}";
string formatted = $"Precio: {price:C}";        // Moneda
string padded = $"{number:D5}";                  // 00042
string aligned = $"{name,10}";                   // Alineado a la derecha en 10 caracteres

// Métodos comunes
s.ToUpper(); s.ToLower();
s.Trim(); s.TrimStart(); s.TrimEnd();
s.Contains("Mundo");
s.StartsWith("Hola"); s.EndsWith("Mundo");
s.Replace("Mundo", "C#");
s.Split(' ');
string.Join(", ", ["a", "b", "c"]);
s.Substring(0, 4);
s.IndexOf("Mundo");
s.Length;

// Comparación
string.Equals("abc", "ABC", StringComparison.OrdinalIgnoreCase);   // true
string.Compare("abc", "abd");    // -1

// StringBuilder — para concatenación intensiva (evita allocations repetidas)
var sb = new StringBuilder();
sb.Append("Hola");
sb.Append(' ').Append("Mundo");
sb.AppendLine();
sb.Insert(0, ">> ");
string result = sb.ToString();

// Raw string literals y verbatim (ver csharp-moderno.md)
string path = @"C:\Users\juan";
string json = """{"key": "value"}""";

// Null/empty checks
string.IsNullOrEmpty(s);
string.IsNullOrWhiteSpace(s);
```

---

## 📅 DateTime, DateOnly, TimeOnly, TimeSpan

```csharp
// DateTime — fecha y hora combinadas
DateTime now = DateTime.Now;           // Hora local
DateTime utcNow = DateTime.UtcNow;     // UTC — preferir para almacenamiento
DateTime specific = new(2026, 7, 23);
DateTime parsed = DateTime.Parse("2026-07-23");

now.AddDays(5); now.AddMonths(1); now.AddYears(-1);
now.Year; now.Month; now.Day; now.DayOfWeek;
now.ToString("yyyy-MM-dd");
now.ToString("dd/MM/yyyy HH:mm:ss");

// DateOnly / TimeOnly (.NET 6+) — separar fecha y hora cuando no se necesita ambas
DateOnly today = DateOnly.FromDateTime(DateTime.Now);
DateOnly birthday = new(1998, 5, 20);
TimeOnly openingTime = new(9, 0);
TimeOnly closingTime = TimeOnly.Parse("18:30");

// TimeSpan — duración
TimeSpan duration = TimeSpan.FromMinutes(90);
TimeSpan diff = DateTime.Now - specific;
duration.TotalHours; duration.Days; duration.Hours; duration.Minutes;

// DateTimeOffset — recomendado cuando importa la zona horaria
DateTimeOffset offset = DateTimeOffset.UtcNow;
DateTimeOffset withZone = new(DateTime.Now, TimeSpan.FromHours(-3));
```

---

## 🔎 Regex

```csharp
using System.Text.RegularExpressions;

// Uso básico
bool isMatch = Regex.IsMatch("hola123", @"^\w+\d+$");
Match match = Regex.Match("Precio: $42.50", @"\$(\d+\.\d+)");
string value = match.Groups[1].Value;   // "42.50"

MatchCollection matches = Regex.Matches("a1 b2 c3", @"\w\d");
foreach (Match m in matches) Console.WriteLine(m.Value);

string replaced = Regex.Replace("hola mundo", @"\s+", "_");

// GeneratedRegex (.NET 7+) — genera el regex en tiempo de compilación, más rápido
public partial class Validators
{
    [GeneratedRegex(@"^[\w.-]+@[\w.-]+\.\w+$")]
    public static partial Regex EmailRegex();
}
bool valid = Validators.EmailRegex().IsMatch("user@example.com");
```

---

## 🔢 Math y Convert

```csharp
Math.Abs(-5);
Math.Max(3, 7); Math.Min(3, 7);
Math.Pow(2, 10);
Math.Sqrt(16);
Math.Round(3.456, 2);       // 3.46
Math.Ceiling(3.1); Math.Floor(3.9);
Math.PI; Math.E;

// Convert — conversiones explícitas entre tipos
int fromString = Convert.ToInt32("42");
double fromInt = Convert.ToDouble(42);
string fromBool = Convert.ToString(true);

// Parse / TryParse — preferido para conversiones desde string con manejo de errores
int.TryParse("42", out int number);
double.TryParse("3.14", out double d);
bool.TryParse("true", out bool b);

// Random (.NET 6+ recomienda Random.Shared para uso simple thread-safe)
int random = Random.Shared.Next(1, 100);
double randomDouble = Random.Shared.NextDouble();
```

---

## 🧮 Guid y HashCode

```csharp
Guid id = Guid.NewGuid();
Guid.TryParse("d3b07384-d9a0-4c8f-8b3a-3c2f1a2e4b5c", out Guid parsed);

// HashCode.Combine — para implementar GetHashCode() correctamente
public override int GetHashCode() => HashCode.Combine(Name, Age);
```

---

## 🛠️ Buenas prácticas

* **`StringBuilder`** en loops de concatenación — `string` es inmutable y cada `+=` crea una nueva instancia.
* **`DateTime.UtcNow`** para almacenamiento y lógica de negocio; convertir a hora local solo para mostrar al usuario.
* **`DateOnly`/`TimeOnly`** cuando el dominio no necesita la otra mitad — evita bugs de zona horaria en fechas puras.
* **`[GeneratedRegex]`** en vez de `new Regex(...)` para patrones usados con frecuencia — se compila en build time.
* **`TryParse` en vez de `Parse`** cuando el input puede ser inválido — evita excepciones para casos esperables.
