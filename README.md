# Developer Cheatsheets

Repositorio de cheatsheets rápidas y prácticas para desarrolladores.  
La idea es tener referencias simples, organizadas por categorías, para consultar comandos, snippets y conceptos comunes sin perder tiempo buscando documentación completa.

---

## Estructura del proyecto

```bash
.
├── developer-tools/
│   ├── docker-cheatsheet.md
│   └── git-cheatsheet.md
│
├── frameworks/
│   ├── android/
│   │   ├── README.md
│   │   └── <tema>/
│   ├── aspnet-core/
│   │   ├── README.md
│   │   └── <tema>/
│   ├── flutter/
│   │   ├── README.md
│   │   └── <tema>/
│   └── springboot/
│       ├── README.md
│       └── <tema>/
│
├── programming-languages/
│   ├── dart-cheatsheet.md
│   └── kotlin-cheatsheet.md
│
├── README.md
└── SOURCES.md
````

---

## Categorías

### Developer Tools

Herramientas utilizadas en el día a día del desarrollo.

* Docker
* Git

---

### Frameworks

Frameworks y tecnologías para desarrollo frontend, backend y mobile.

* Android
* ASP.NET Core
* Flutter
* Spring Boot

---

### Programming Languages

Lenguajes de programación y ecosistemas relacionados.

* Dart
* Kotlin

---

## Objetivo

Este repositorio busca:

* Tener referencias rápidas y fáciles de leer.
* Centralizar snippets y comandos comunes.
* Evitar perder tiempo buscando en documentación extensa.
* Servir como material de estudio y repaso.

---

## Contribuciones

Las contribuciones son bienvenidas.

Puedes colaborar agregando:

* Nuevos cheatsheets
* Correcciones
* Mejores prácticas
* Ejemplos útiles
* Organización de categorías

### Pasos

1. Haz un fork del repositorio
2. Crea una rama nueva
3. Realiza tus cambios
4. Envía un Pull Request

---

## Convención de nombres

Los archivos siguen esta estructura:

```bash
<technology>-cheatsheet.md
```

Ejemplos:

```bash
git-cheatsheet.md
docker-cheatsheet.md
```

Para frameworks (que suelen tener muchas subsecciones), se usa en cambio una carpeta por framework con subcarpetas por tema y un `README.md` con el índice:

```bash
frameworks/<framework>/README.md
frameworks/<framework>/<tema>/<tema>.md
```

---

## Fuentes

[`SOURCES.md`](SOURCES.md) registra la documentación oficial consultada para verificar y actualizar cada cheatsheet (versiones, APIs vigentes, deprecaciones). Solo se citan fuentes oficiales — nada de blogs ni artículos de terceros.

---

## Recomendaciones

Cada cheatsheet debería incluir:

* Descripción breve
* Instalación (si aplica)
* Comandos básicos
* Ejemplos rápidos
* Tips útiles
* Buenas prácticas

---

## Licencia

MIT License.
