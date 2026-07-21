# Spring Boot — Cheatsheet (Java + Maven + Lombok)

Cheatsheet de Spring Boot organizado por temas. Cada carpeta contiene el/los `.md` de su categoría.

---

## 📑 Índice

### Base
- [pom.xml, estructura de proyecto, application.properties, Main class y comandos Maven](base/base.md)

### Web / API REST
- [RestController, ResponseEntity, manejo de excepciones y validación (Bean Validation)](web-api/web-api.md)

### Lombok
- [Anotaciones esenciales](lombok/lombok.md)

### DTOs y Mappers
- [DTOs con record y Lombok, MapStruct](dtos-mappers/dtos-mappers.md)

### Inyección de dependencias
- [@Component/@Service/@Repository y @Configuration/@Bean](inyeccion-dependencias/inyeccion-dependencias.md)

### Persistencia
- [Spring Data JPA — entidades, repositorios, paginación, transacciones y Flyway](persistencia/persistencia.md)

### Seguridad
- [Spring Security, JWT y autorización](seguridad/seguridad.md)

### Configuración avanzada
- [@ConfigurationProperties, @Profile/@Conditional y propiedades por ambiente](configuracion-avanzada/configuracion-avanzada.md)

### Features
- [Caché, Async, Scheduling, Eventos, Mail y Actuator](features/features.md)

### Testing
- [@SpringBootTest, @WebMvcTest, @DataJpaTest, MockMvc, Mockito y Testcontainers](testing/testing.md)

### DevOps
- [Docker — Dockerfile y docker-compose.yml](devops/devops.md)

---

## 💡 Buenas prácticas y patrones

### Arquitectura por capas

```
Controller  → recibe HTTP, delega al Service, devuelve ResponseEntity
Service     → lógica de negocio, orquesta, usa Repository
Repository  → acceso a datos, sin lógica de negocio
Entity      → modelo de persistencia, solo JPA
DTO         → transferencia de datos entre capas (record o Lombok)
Mapper      → convierte entre Entity y DTO (MapStruct)
Exception   → excepciones de dominio bien tipadas
```

### Reglas clave

```java
// ✅ Usar interface para Services — facilita testing y desacoplamiento
public interface ProductService {
    ProductResponse findById(Long id);
    ProductResponse create(CreateProductRequest request);
}

@Service
public class ProductServiceImpl implements ProductService { ... }

// ✅ Siempre open-in-view=false y @Transactional(readOnly=true) en lecturas
@Transactional(readOnly = true)
public List<ProductResponse> findAll() { ... }

// ✅ Nunca exponer entidades JPA en los controllers — siempre usar DTOs
// ❌ return productRepository.findAll();
// ✅ return productRepository.findAll().stream().map(productMapper::toResponse).toList();

// ✅ Inicializar colecciones en las entidades para evitar NullPointerException
private List<ProductImage> images = new ArrayList<>();
private Set<String> tags = new HashSet<>();

// ✅ Fetch LAZY por defecto, EAGER solo cuando siempre se necesita
@ManyToOne(fetch = FetchType.LAZY)     // ✅ Default recomendado
@OneToMany(fetch = FetchType.LAZY)     // ✅ Default recomendado

// ✅ Evitar N+1 con @EntityGraph o JOIN FETCH
@EntityGraph(attributePaths = {"category", "images"})
Optional<Product> findByIdWithDetails(Long id);

// ✅ Validar en el Service, no solo en el Controller
// El Controller valida el formato (Bean Validation)
// El Service valida las reglas de negocio

// ✅ Manejar solo RuntimeException en @Transactional para rollback automático
// Las checked exceptions NO hacen rollback a menos que especifiques rollbackFor

// ✅ Usar @Slf4j de Lombok — no crear loggers manualmente
// ❌ private static final Logger log = LoggerFactory.getLogger(MyClass.class);
// ✅ @Slf4j en la clase y usar log.info(...)

// ✅ Nunca loguear datos sensibles
log.info("Login para usuario: {}", email);       // ✅
log.info("Password: {}", password);               // ❌ NUNCA

// ✅ Usar paginación siempre — nunca findAll() en producción sin límite
// ❌ productRepository.findAll()
// ✅ productRepository.findAll(pageable)

// ✅ Externalizar toda la configuración en application.properties / variables de entorno
// ❌ private String secret = "hardcoded-secret";
// ✅ @Value("${app.jwt.secret}") private String secret;

// ✅ Graceful shutdown configurado (spring.lifecycle.timeout-per-shutdown-phase=30s)

// ✅ Versionar la API desde el inicio
// /api/v1/products
// /api/v2/products (breaking changes)

// ✅ Usar HTTP correcto:
// POST   → crear       → 201 Created
// GET    → leer        → 200 OK
// PUT    → reemplazar  → 200 OK
// PATCH  → actualizar  → 200 OK
// DELETE → eliminar    → 204 No Content
// GET inexistente      → 404 Not Found
// POST duplicado       → 409 Conflict
// POST inválido        → 400 Bad Request
```
