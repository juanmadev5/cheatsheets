# Spring Boot — Persistencia

---

## 🏗️ Spring Data JPA — entidades

```java
// BaseEntity — campos comunes heredados por todas las entidades
@MappedSuperclass
@Getter
@Setter
@EntityListeners(AuditingEntityListener.class)  // Auditoría automática
public abstract class BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @CreatedDate
    @Column(nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    @Column(nullable = false)
    private LocalDateTime updatedAt;

    @CreatedBy
    @Column(updatable = false)
    private String createdBy;

    @LastModifiedBy
    private String updatedBy;

    @Version                            // Optimistic locking
    private Long version;
}

// Activar auditoría en el main
@SpringBootApplication
@EnableJpaAuditing(auditorAwareRef = "auditorProvider")
public class MiApiApplication { ... }

@Component
public class AuditorProvider implements AuditorAware<String> {
    @Override
    public Optional<String> getCurrentAuditor() {
        return Optional.ofNullable(SecurityContextHolder.getContext().getAuthentication())
            .map(Authentication::getName);
    }
}

// Entidad completa
@Entity
@Table(name = "products", indexes = {
    @Index(name = "idx_product_sku", columnList = "sku", unique = true),
    @Index(name = "idx_product_category", columnList = "category_id"),
    @Index(name = "idx_product_active", columnList = "active"),
})
@Getter
@Setter
@NoArgsConstructor
@ToString(exclude = {"category", "orderItems", "reviews"})
@EqualsAndHashCode(onlyExplicitlyIncluded = true, callSuper = false)
public class Product extends BaseEntity {

    @EqualsAndHashCode.Include
    @Column(nullable = false, unique = true, length = 50)
    private String sku;

    @Column(nullable = false, length = 200)
    private String name;

    @Column(columnDefinition = "TEXT")
    private String description;

    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal price;

    @Column(nullable = false)
    private Integer stock = 0;

    @Column(nullable = false)
    private boolean active = true;

    // Enum mapeado como String
    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 30)
    private ProductStatus status = ProductStatus.DRAFT;

    // Colección de tipos simples
    @ElementCollection
    @CollectionTable(name = "product_tags",
        joinColumns = @JoinColumn(name = "product_id"))
    @Column(name = "tag")
    private Set<String> tags = new HashSet<>();

    // Mapa de atributos
    @ElementCollection
    @CollectionTable(name = "product_attributes",
        joinColumns = @JoinColumn(name = "product_id"))
    @MapKeyColumn(name = "attr_key")
    @Column(name = "attr_value")
    private Map<String, String> attributes = new HashMap<>();

    // Relación ManyToOne — FK en esta tabla
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "category_id", nullable = false)
    private Category category;

    // Relación OneToMany — colección
    @OneToMany(mappedBy = "product",
        cascade = CascadeType.ALL,
        orphanRemoval = true,
        fetch = FetchType.LAZY)
    private List<ProductImage> images = new ArrayList<>();

    // Relación ManyToMany
    @ManyToMany
    @JoinTable(name = "product_tags_rel",
        joinColumns = @JoinColumn(name = "product_id"),
        inverseJoinColumns = @JoinColumn(name = "tag_id"))
    private Set<Tag> tagEntities = new HashSet<>();

    // Columna JSON (PostgreSQL)
    @JdbcTypeCode(SqlTypes.JSON)
    @Column(columnDefinition = "jsonb")
    private Map<String, Object> metadata;

    // Métodos de gestión de relaciones bidireccionales
    public void addImage(ProductImage image) {
        images.add(image);
        image.setProduct(this);
    }

    public void removeImage(ProductImage image) {
        images.remove(image);
        image.setProduct(null);
    }
}

// Enum de dominio
public enum ProductStatus {
    DRAFT("Borrador"),
    ACTIVE("Activo"),
    INACTIVE("Inactivo"),
    DISCONTINUED("Discontinuado");

    private final String label;
    ProductStatus(String label) { this.label = label; }
    public String getLabel() { return label; }
}

// Entidad de usuario
@Entity
@Table(name = "users")
@Getter
@Setter
@NoArgsConstructor
@ToString(exclude = "password")
@EqualsAndHashCode(onlyExplicitlyIncluded = true, callSuper = false)
public class User extends BaseEntity implements UserDetails {

    @EqualsAndHashCode.Include
    @Column(nullable = false, unique = true, length = 150)
    private String email;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    @ToString.Exclude
    private String password;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private Role role = Role.USER;

    @Column(nullable = false)
    private boolean enabled = true;

    @Column(nullable = false)
    private boolean accountNonLocked = true;

    // UserDetails — Spring Security
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(new SimpleGrantedAuthority("ROLE_" + role.name()));
    }

    @Override
    public String getUsername() { return email; }

    @Override
    public boolean isAccountNonExpired() { return true; }

    @Override
    public boolean isAccountNonLocked() { return accountNonLocked; }

    @Override
    public boolean isCredentialsNonExpired() { return true; }
}

public enum Role { USER, ADMIN, MODERATOR }
```

---

## 🔍 Repositorios

```java
// JpaRepository — CRUD + paginación incluidos
public interface ProductRepository extends JpaRepository<Product, Long>,
        JpaSpecificationExecutor<Product> {  // Para Specification

    // Derived queries — Spring genera el SQL automáticamente por el nombre del método
    Optional<Product> findBySku(String sku);
    boolean existsBySku(String sku);
    List<Product> findByActiveTrue();
    List<Product> findByCategoryName(String categoryName);     // Acceso anidado
    List<Product> findByPriceBetween(BigDecimal min, BigDecimal max);
    List<Product> findByNameContainingIgnoreCase(String name);
    List<Product> findByStockLessThan(int stock);
    long countByActiveTrue();
    void deleteByActiveFalse();
    List<Product> findTop10ByActiveTrueOrderByCreatedAtDesc();

    // Múltiples condiciones
    List<Product> findByCategoryNameAndActiveTrueAndPriceLessThan(
        String category, BigDecimal maxPrice);

    // @Query — JPQL personalizada
    @Query("SELECT p FROM Product p WHERE p.active = true AND " +
           "(:name IS NULL OR LOWER(p.name) LIKE LOWER(CONCAT('%', :name, '%'))) AND " +
           "(:category IS NULL OR p.category.name = :category) AND " +
           "(:minPrice IS NULL OR p.price >= :minPrice) AND " +
           "(:maxPrice IS NULL OR p.price <= :maxPrice)")
    Page<Product> searchProducts(
        @Param("name") String name,
        @Param("category") String category,
        @Param("minPrice") BigDecimal minPrice,
        @Param("maxPrice") BigDecimal maxPrice,
        Pageable pageable);

    // @Query con JOIN FETCH — evitar N+1
    @Query("SELECT DISTINCT p FROM Product p " +
           "LEFT JOIN FETCH p.category " +
           "LEFT JOIN FETCH p.images " +
           "WHERE p.id = :id")
    Optional<Product> findByIdWithDetails(@Param("id") Long id);

    // Query nativa (SQL puro)
    @Query(value = """
        SELECT p.*, c.name as category_name
        FROM products p
        JOIN categories c ON p.category_id = c.id
        WHERE p.active = true
        ORDER BY p.created_at DESC
        LIMIT :limit
        """, nativeQuery = true)
    List<Object[]> findActiveProductsNative(@Param("limit") int limit);

    // @Modifying — UPDATE o DELETE
    @Modifying
    @Transactional
    @Query("UPDATE Product p SET p.stock = p.stock - :quantity WHERE p.id = :id AND p.stock >= :quantity")
    int decrementStock(@Param("id") Long id, @Param("quantity") int quantity);

    @Modifying
    @Transactional
    @Query("UPDATE Product p SET p.active = false WHERE p.id IN :ids")
    int deactivateByIds(@Param("ids") List<Long> ids);

    // Proyecciones — solo traer campos necesarios
    List<ProductSummary> findByActiveTrue();

    // Interfaz de proyección
    interface ProductSummary {
        Long getId();
        String getName();
        BigDecimal getPrice();
        String getCategoryName();    // Acceso anidado con getCategory().getName()
    }

    // Proyección con @Value SpEL
    interface ProductLabel {
        @Value("#{target.name + ' - $' + target.price}")
        String getLabel();
    }
}

// Specification — queries dinámicas con criterios combinables
public class ProductSpecification {

    public static Specification<Product> hasName(String name) {
        return (root, query, cb) -> name == null ? null :
            cb.like(cb.lower(root.get("name")), "%" + name.toLowerCase() + "%");
    }

    public static Specification<Product> hasCategory(String category) {
        return (root, query, cb) -> category == null ? null :
            cb.equal(root.join("category").get("name"), category);
    }

    public static Specification<Product> priceBetween(BigDecimal min, BigDecimal max) {
        return (root, query, cb) -> {
            if (min != null && max != null)
                return cb.between(root.get("price"), min, max);
            if (min != null) return cb.greaterThanOrEqualTo(root.get("price"), min);
            if (max != null) return cb.lessThanOrEqualTo(root.get("price"), max);
            return null;
        };
    }

    public static Specification<Product> isActive() {
        return (root, query, cb) -> cb.isTrue(root.get("active"));
    }
}

// Uso de Specification en Service
public Page<ProductResponse> search(String name, String category,
        BigDecimal minPrice, BigDecimal maxPrice, Pageable pageable) {

    var spec = Specification.where(ProductSpecification.isActive())
        .and(ProductSpecification.hasName(name))
        .and(ProductSpecification.hasCategory(category))
        .and(ProductSpecification.priceBetween(minPrice, maxPrice));

    return productRepository.findAll(spec, pageable)
        .map(productMapper::toResponse);
}
```

---

## 📄 Paginación y ordenamiento

```java
// Pageable en el controller
@GetMapping
public ResponseEntity<PageResponse<ProductResponse>> findAll(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size,
        @RequestParam(defaultValue = "createdAt,desc") String[] sort) {

    // Parsear sort: "field,direction"
    var orders = Arrays.stream(sort)
        .map(s -> s.split(","))
        .map(arr -> new Sort.Order(
            arr.length > 1 ? Sort.Direction.fromString(arr[1]) : Sort.Direction.ASC,
            arr[0]))
        .toList();

    var pageable = PageRequest.of(page, Math.min(size, 100), Sort.by(orders));
    return ResponseEntity.ok(productService.findAll(pageable));
}

// Alternativamente — usar PageableHandlerMethodArgumentResolver (automático)
// Spring resuelve Pageable directamente del request con ?page=0&size=10&sort=name,asc
@GetMapping
public ResponseEntity<Page<ProductResponse>> findAll(
        @PageableDefault(size = 20, sort = "createdAt", direction = Sort.Direction.DESC)
        Pageable pageable) {
    return ResponseEntity.ok(productService.findAll(pageable).map(productMapper::toResponse));
}

// En application.properties:
// spring.data.web.pageable.default-page-size=20
// spring.data.web.pageable.max-page-size=100
// spring.data.web.pageable.one-indexed-parameters=false

// Slice — paginación sin contar el total (más eficiente para scroll infinito)
public interface ProductRepository extends JpaRepository<Product, Long> {
    Slice<Product> findByActiveTrue(Pageable pageable);
}
```

---

## 🔄 Transacciones

```java
// @Transactional — básico
@Service
public class ProductService {

    // readOnly=true para lecturas — optimización de Hibernate
    @Transactional(readOnly = true)
    public ProductResponse findById(Long id) {
        return productRepository.findById(id)
            .map(productMapper::toResponse)
            .orElseThrow(() -> new ResourceNotFoundException("Product", id));
    }

    // Transacción de escritura — rollback automático en RuntimeException
    @Transactional
    public ProductResponse create(CreateProductRequest request) {
        // Si lanza RuntimeException → rollback automático
        var product = productMapper.toEntity(request);
        return productMapper.toResponse(productRepository.save(product));
    }

    // Rollback en excepciones específicas (incluyendo checked)
    @Transactional(rollbackFor = {BusinessException.class, IOException.class})
    public void processOrder(Long orderId) throws IOException { ... }

    // No hacer rollback en una excepción específica
    @Transactional(noRollbackFor = ResourceNotFoundException.class)
    public void softDelete(Long id) { ... }

    // Timeout — abortar si tarda más de N segundos
    @Transactional(timeout = 30)
    public void heavyOperation() { ... }
}

// Propagación de transacciones
// REQUIRED (default) — usa la existente o crea nueva
// REQUIRES_NEW — siempre crea nueva (suspende la actual)
// NESTED — transacción anidada con savepoint
// SUPPORTS — usa la existente si hay, sino sin transacción
// NOT_SUPPORTED — siempre sin transacción
// MANDATORY — exige transacción existente o lanza excepción
// NEVER — lanza excepción si hay transacción activa

@Transactional(propagation = Propagation.REQUIRES_NEW)
public void auditLog(String action, Long entityId) {
    // Siempre se guarda, incluso si la transacción principal hace rollback
    auditRepository.save(new AuditLog(action, entityId));
}

// Niveles de aislamiento
// DEFAULT — el del motor de BD
// READ_UNCOMMITTED — ve cambios no commiteados (dirty reads)
// READ_COMMITTED — ve solo cambios commiteados (default PostgreSQL)
// REPEATABLE_READ — misma lectura en la transacción
// SERIALIZABLE — máximo aislamiento

@Transactional(isolation = Isolation.SERIALIZABLE)
public void criticalOperation() { ... }

// Transacciones programáticas (cuando el aspecto no funciona, ej: métodos self-invocados)
@Service
@RequiredArgsConstructor
public class ProductService {

    private final TransactionTemplate transactionTemplate;

    public ProductResponse createWithTemplate(CreateProductRequest request) {
        return transactionTemplate.execute(status -> {
            try {
                var product = productMapper.toEntity(request);
                return productMapper.toResponse(productRepository.save(product));
            } catch (Exception e) {
                status.setRollbackOnly();
                throw e;
            }
        });
    }
}
```

---

## 🗃️ Migraciones — Flyway

```sql
-- V1__create_users_table.sql
CREATE TABLE users (
    id          BIGSERIAL PRIMARY KEY,
    email       VARCHAR(150) NOT NULL UNIQUE,
    name        VARCHAR(200) NOT NULL,
    password    VARCHAR(255) NOT NULL,
    role        VARCHAR(20)  NOT NULL DEFAULT 'USER',
    enabled     BOOLEAN      NOT NULL DEFAULT true,
    account_non_locked BOOLEAN NOT NULL DEFAULT true,
    created_at  TIMESTAMP    NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMP    NOT NULL DEFAULT NOW(),
    created_by  VARCHAR(150),
    updated_by  VARCHAR(150),
    version     BIGINT       NOT NULL DEFAULT 0
);

CREATE INDEX idx_users_email ON users(email);
```

```sql
-- V2__create_categories_and_products.sql
CREATE TABLE categories (
    id          BIGSERIAL PRIMARY KEY,
    name        VARCHAR(100) NOT NULL UNIQUE,
    slug        VARCHAR(100) NOT NULL UNIQUE,
    description TEXT,
    active      BOOLEAN      NOT NULL DEFAULT true,
    created_at  TIMESTAMP    NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMP    NOT NULL DEFAULT NOW(),
    version     BIGINT       NOT NULL DEFAULT 0
);

CREATE TABLE products (
    id          BIGSERIAL PRIMARY KEY,
    sku         VARCHAR(50)    NOT NULL UNIQUE,
    name        VARCHAR(200)   NOT NULL,
    description TEXT,
    price       NUMERIC(10,2)  NOT NULL CHECK (price >= 0),
    stock       INTEGER        NOT NULL DEFAULT 0 CHECK (stock >= 0),
    active      BOOLEAN        NOT NULL DEFAULT true,
    status      VARCHAR(30)    NOT NULL DEFAULT 'DRAFT',
    metadata    JSONB,
    category_id BIGINT         NOT NULL REFERENCES categories(id),
    created_at  TIMESTAMP      NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMP      NOT NULL DEFAULT NOW(),
    created_by  VARCHAR(150),
    updated_by  VARCHAR(150),
    version     BIGINT         NOT NULL DEFAULT 0
);

CREATE INDEX idx_products_sku      ON products(sku);
CREATE INDEX idx_products_category ON products(category_id);
CREATE INDEX idx_products_active   ON products(active);
CREATE INDEX idx_products_status   ON products(status);
CREATE INDEX idx_products_metadata ON products USING GIN(metadata);
```

```sql
-- V3__seed_initial_data.sql
INSERT INTO categories (name, slug) VALUES
    ('Electrónica', 'electronica'),
    ('Ropa', 'ropa'),
    ('Alimentos', 'alimentos');

INSERT INTO users (email, name, password, role) VALUES
    ('admin@miapp.com', 'Administrador',
     '$2a$12$...hash_de_la_password...', 'ADMIN');
```

```java
// Reparar migration fallida (development)
// flyway.repair() — reparar checksum de migrations
@Bean
public FlywayMigrationStrategy flywayMigrationStrategy() {
    return flyway -> {
        flyway.repair();   // Solo en dev
        flyway.migrate();
    };
}
```

