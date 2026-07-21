# Spring Boot — Lombok

---

## 🏷️ Lombok — anotaciones esenciales

```java
// @Data = @Getter + @Setter + @RequiredArgsConstructor + @ToString + @EqualsAndHashCode
@Data
public class ProductDto {
    private Long id;
    private String name;
    private BigDecimal price;
}

// @Getter / @Setter — solo getters o solo setters
@Getter
@Setter
public class UserDto { ... }

// @Getter a nivel de campo
public class Config {
    @Getter private final String apiUrl;
    @Getter @Setter private int timeout;
}

// @NoArgsConstructor — constructor sin argumentos
// @AllArgsConstructor — constructor con todos los campos
// @RequiredArgsConstructor — constructor con campos final y @NonNull
@NoArgsConstructor
@AllArgsConstructor
@RequiredArgsConstructor
public class Product {
    @NonNull private String name;   // Incluido en @RequiredArgsConstructor
    private String description;     // NO incluido (no es final ni @NonNull)
    private final String sku;       // Incluido (es final)
}

// @Builder — patrón Builder
@Builder
@Getter
public class ProductResponse {
    private Long id;
    private String name;
    private BigDecimal price;
    @Builder.Default                // Valor por defecto en el builder
    private boolean active = true;
}

// Uso
var product = ProductResponse.builder()
    .id(1L)
    .name("Laptop")
    .price(new BigDecimal("999.99"))
    .build();

// @Builder con herencia — usar @SuperBuilder
@SuperBuilder
@Getter
public class BaseEntity {
    private Long id;
    private LocalDateTime createdAt;
}

@SuperBuilder
@Getter
public class Product extends BaseEntity {
    private String name;
    private BigDecimal price;
}

// @Value — clase inmutable (todos los campos final)
// = @Getter + @AllArgsConstructor + @ToString + @EqualsAndHashCode + fields final
@Value
public class Money {
    BigDecimal amount;
    String currency;
}

// @Slf4j — logger (también: @Log, @Log4j2, @CommonsLog)
@Slf4j
@Service
public class ProductService {
    public void create(CreateProductRequest req) {
        log.debug("Creando producto: {}", req.getName());
        log.info("Producto creado con ID: {}", id);
        log.warn("Stock bajo para producto: {}", id);
        log.error("Error al crear producto", exception);
    }
}

// @ToString — personalizar toString
@ToString(exclude = {"password", "token"})      // Excluir campos sensibles
@ToString(onlyExplicitlyIncluded = true)
public class User {
    @ToString.Include private Long id;
    @ToString.Include private String name;
    private String password;                     // No incluido
}

// @EqualsAndHashCode — controlar equals/hashCode
@EqualsAndHashCode(onlyExplicitlyIncluded = true)
public class Product {
    @EqualsAndHashCode.Include
    private String sku;             // Solo SKU define igualdad
    private String name;
    private BigDecimal price;
}

// @With — generar withX() para copias inmutables
@Value
@With
public class Config {
    String apiUrl;
    int timeout;
    boolean debug;
}

var config = new Config("https://api.com", 30, false);
var debugConfig = config.withDebug(true);   // Copia con debug=true

// @SneakyThrows — relanzar checked exceptions como unchecked
@SneakyThrows
public String readFile(String path) {
    return Files.readString(Path.of(path));  // IOException manejada automáticamente
}

// @Cleanup — cerrar recursos automáticamente
public void processFile(String path) throws Exception {
    @Cleanup var stream = new FileInputStream(path);
    // stream.close() se llama automáticamente al salir del scope
}

// @NonNull — verificación null en tiempo de ejecución
public void setName(@NonNull String name) {
    this.name = name;   // Lanza NullPointerException si name es null
}

// Entidad JPA con Lombok — configuración correcta
@Entity
@Table(name = "products")
@Getter
@Setter
@NoArgsConstructor
@ToString(exclude = {"category", "orderItems"})  // Evitar ciclos en relaciones
@EqualsAndHashCode(onlyExplicitlyIncluded = true)
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @EqualsAndHashCode.Include
    private Long id;

    private String name;
    private BigDecimal price;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "category_id")
    private Category category;
}
// NOTA: en entidades JPA NUNCA usar @Data ni @EqualsAndHashCode sin configurar
// porque genera equals/hashCode basado en todos los campos incluyendo las
// colecciones lazy, lo que puede causar LazyInitializationException
```

