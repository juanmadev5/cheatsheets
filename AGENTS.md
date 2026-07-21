# Guía de Contribución y Reglas para Agentes de IA

Este documento define las normas y flujos de trabajo para cualquier Agente de IA o desarrollador humano que interactúe, consulte o mantenga este repositorio de cheatsheets.

---

## 📄 Plantilla Estándar para Cheatsheets

Todos los cheatsheets creados o actualizados deben seguir esta estructura visual clara, práctica y enfocada en código y comandos aplicables:

```markdown
# <Tecnología / Framework> — <Tema o Categoría>

---

## 📌 <Nombre del Tema / Sección Principal>

<Breve párrafo introductorio de 1 o 2 líneas explicando qué cubre esta sección.>

### Tabla de resumen o conceptos clave (si aplica)

| Concepto / Tipo | Descripción | Ejemplo / Uso |
|---|---|---|
| **Concepto 1** | Explicación breve | `Comando / Sintaxis` |
| **Concepto 2** | Explicación breve | `Comando / Sintaxis` |

---

### <Subtema o Caso de Uso Concreto>

<Explicación técnica en español de 1 línea.>

```<lenguaje>
// Código en inglés con comentarios breves en español
fun ejemplo(): String {
    return "Ejemplo autocontenido"
}
```

```bash
# Comandos de terminal en inglés con comentarios en español
docker ps --format "table {{.ID}}\t{{.Names}}"
```

---

## 🛠️ <Comandos / Opciones Útiles / Siguientes Pasos>

* **`opcion_1`**: Explicación breve en español.
* **`opcion_2`**: Explicación breve en español.
```

### 💡 Pautas de Diseño para los Cheatsheets:
1. **Atractivo y legible a primera vista:** Usa tablas comparativas, emojis en los encabezados (`📂`, `📨`, `🧱`, `⚙️`) y separadores de sección (`---`) para cortar bloques densos de texto.
2. **Código completo y funcional:** Los bloques de código no deben ser fragmentos ambiguos. Deben incluir context-imports o estructuras que funcionen directamente al copiar y pegar.
3. **Bilingüismo:** Explicaciones y comentarios en **español**, comandos y código en **inglés**.

---

## 📁 Estructura del Repositorio y Subcarpetas

Las tecnologías dentro de `frameworks/` **siempre** deben dividirse en subcarpetas siguiendo el patrón estricto:

```
frameworks/<framework>/<tema>/<tema>.md
```

*Ejemplo:* `frameworks/android/almacenamiento/almacenamiento.md`

---

## 📌 Flujo para Actualizar los Índices (`README.md`)

1. **`README.md` del Framework (`frameworks/<framework>/README.md`):**
   * Al agregar un nuevo tema o cheatsheet dentro de un framework, actualiza inmediatamente el índice local agregando el enlace correspondiente:
     ```markdown
     * [Almacenamiento](almacenamiento/almacenamiento.md) — Breve explicación de 1 línea.
     ```

2. **`README.md` Principal (`/README.md`):**
   * Si agregas un **nuevo framework**, **herramienta** o **lenguaje**, añádelo a la lista correspondiente dentro de la sección `## Categorías`.
   * Si creas una nueva categoría superior, actualiza también el árbol de directorios en la sección de estructura del proyecto.

---

## 🔄 Flujo para Actualizar `SOURCES.md`

Para mantener la calidad técnica y evitar enlaces no verificados:

1. **Solo Fuentes Oficiales:** Prohibido citar Medium, dev.to, blogs de terceros o StackOverflow. Solo documentación oficial (`developer.android.com`, `docs.flutter.dev`, `learn.microsoft.com`, repositorios de GitHub oficiales).
2. **Ubicación de la cita:** 
   * Si la tecnología/framework ya existe en `SOURCES.md`, agrega la nueva URL bajo la subsección o categoría correspondiente.
   * Si es un framework nuevo o tema relevante, crea un nuevo encabezado `## <Tecnología>` e incluye la fecha de investigación (ejemplo: `_Investigado: julio 2026_`).
3. **Formato del ítem:**
   ```markdown
   - https://url-oficial.com/docs — Breve nota de qué se verificó (versión, API, deprecación, etc.)
   ```

---

## 🤖 Reglas de Consulta para Agentes de IA (Querying / RAG)

Cuando una IA lea este repositorio para responder dudas o realizar modificaciones:
1. **Índices de búsqueda:** Consultar primero `frameworks/<framework>/README.md` o el `README.md` raíz para identificar la ruta exacta del tema.
2. **Validación de vigencia:** Consultar `SOURCES.md` para verificar qué versiones, APIS o estándares están tomados de referencia y asegurar coherencia con las versiones actuales.
