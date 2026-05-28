# Spring Boot — Cheatsheet (Java + Maven + Lombok)

---

## 📑 Índice

### Base
- [pom.xml — dependencias esenciales](#️-pomxml--dependencias-esenciales)
- [Estructura de proyecto](#-estructura-de-proyecto)
- [application.properties — configuración completa](#️-applicationproperties--configuración-completa)
- [Main class y runners](#-main-class-y-runners)

### Web / API REST
- [RestController — GET, POST, PUT, PATCH, DELETE](#-restcontroller)
- [ResponseEntity y respuestas estandarizadas](#-responseentity-y-respuestas-estandarizadas)
- [Manejo global de excepciones — @ControllerAdvice](#️-manejo-global-de-excepciones)
- [Validación — Bean Validation + custom validators](#-validación--bean-validation)

### Lombok
- [Lombok — anotaciones esenciales](#-lombok--anotaciones-esenciales)

### DTOs y Mappers
- [DTOs con record y Lombok](#-dtos-con-record-y-lombok)
- [MapStruct — mapeo automático](#️-mapstruct--mapeo-automático)

### Inyección de dependencias
- [Inyección de dependencias — @Component, @Service, @Repository](#-inyección-de-dependencias)
- [Configuración — @Configuration, @Bean, @Qualifier](#️-configuración--beans)

### Persistencia
- [Spring Data JPA — entidades y relaciones](#️-spring-data-jpa--entidades)
- [Repositorios — JpaRepository, queries, Specification](#-repositorios)
- [Paginación y ordenamiento — Pageable](#-paginación-y-ordenamiento)
- [Transacciones — @Transactional, propagación, isolation](#-transacciones)
- [Migraciones — Flyway](#️-migraciones--flyway)

### Seguridad
- [Spring Security — SecurityFilterChain, JWT](#-spring-security)
- [JWT — filter, provider, utilidades](#-jwt--filter-y-utilidades)
- [Autorización — roles, anotaciones de método](#-autorización)

### Configuración avanzada
- [@ConfigurationProperties](#️-configurationproperties)
- [@Profile y @Conditional](#-profile-y-conditional)
- [Propiedades por ambiente](#-propiedades-por-ambiente)

### Features
- [Caché — @Cacheable, @CacheEvict, Redis](#-caché)
- [Async — @Async, CompletableFuture](#-async)
- [Scheduling — @Scheduled](#️-scheduling)
- [Eventos — ApplicationEvent, @EventListener](#-eventos)
- [Mail — JavaMailSender](#-mail)
- [Actuator — health, metrics, endpoints](#-actuator)

### Testing
- [Testing — @SpringBootTest, @WebMvcTest, @DataJpaTest](#-testing)
- [MockMvc, Mockito, Testcontainers](#-mockmvc-mockito-y-testcontainers)

### DevOps
- [Docker — Dockerfile y docker-compose.yml](#-docker)
- [Buenas prácticas y patrones](#-buenas-prácticas-y-patrones)

---

## ⚙️ pom.xml — dependencias esenciales

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <!-- Spring Boot Parent — gestiona versiones de todas las dependencias -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.4.1</version>
        <relativePath/>
    </parent>

    <groupId>com.miempresa</groupId>
    <artifactId>mi-api</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
    <name>mi-api</name>
    <description>API REST con Spring Boot</description>

    <properties>
        <java.version>21</java.version>
        <mapstruct.version>1.6.3</mapstruct.version>
        <lombok.version>1.18.36</lombok.version>
        <testcontainers.version>1.20.4</testcontainers.version>
        <jjwt.version>0.12.6</jjwt.version>
    </properties>

    <dependencies>

        <!-- ===== WEB ===== -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Validación — Bean Validation / Hibernate Validator -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <!-- ===== PERSISTENCIA ===== -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <!-- PostgreSQL -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- Flyway — migraciones de base de datos -->
        <dependency>
            <groupId>org.flywaydb</groupId>
            <artifactId>flyway-core</artifactId>
        </dependency>
        <dependency>
            <groupId>org.flywaydb</groupId>
            <artifactId>flyway-database-postgresql</artifactId>
        </dependency>

        <!-- ===== SEGURIDAD ===== -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>

        <!-- JWT -->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-api</artifactId>
            <version>${jjwt.version}</version>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-impl</artifactId>
            <version>${jjwt.version}</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-jackson</artifactId>
            <version>${jjwt.version}</version>
            <scope>runtime</scope>
        </dependency>

        <!-- ===== CACHÉ ===== -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

        <!-- ===== MAIL ===== -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-mail</artifactId>
        </dependency>

        <!-- ===== ACTUATOR ===== -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <!-- ===== LOMBOK ===== -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
            <optional>true</optional>
        </dependency>

        <!-- ===== MAPSTRUCT ===== -->
        <dependency>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct</artifactId>
            <version>${mapstruct.version}</version>
        </dependency>

        <!-- ===== DEVTOOLS (solo desarrollo) ===== -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>

        <!-- ===== TESTING ===== -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!-- H2 para tests en memoria -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>test</scope>
        </dependency>

        <!-- Testcontainers -->
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>${testcontainers.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>postgresql</artifactId>
            <version>${testcontainers.version}</version>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <build>
        <plugins>

            <!-- Spring Boot Maven Plugin -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <!-- Excluir Lombok del JAR final -->
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>

            <!-- Maven Compiler Plugin — Lombok + MapStruct -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                    <!-- CRÍTICO: Lombok debe ir ANTES que MapStruct -->
                    <annotationProcessorPaths>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                            <version>${lombok.version}</version>
                        </path>
                        <path>
                            <groupId>org.mapstruct</groupId>
                            <artifactId>mapstruct-processor</artifactId>
                            <version>${mapstruct.version}</version>
                        </path>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok-mapstruct-binding</artifactId>
                            <version>0.2.0</version>
                        </path>
                    </annotationProcessorPaths>
                    <compilerArgs>
                        <arg>-Amapstruct.defaultComponentModel=spring</arg>
                    </compilerArgs>
                </configuration>
            </plugin>

            <!-- Surefire — tests unitarios -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <configuration>
                    <includes>
                        <include>**/*Test.java</include>
                        <include>**/*Tests.java</include>
                    </includes>
                </configuration>
            </plugin>

            <!-- Failsafe — tests de integración -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-failsafe-plugin</artifactId>
                <executions>
                    <execution>
                        <goals>
                            <goal>integration-test</goal>
                            <goal>verify</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

        </plugins>
    </build>

    <!-- Perfiles por ambiente -->
    <profiles>
        <profile>
            <id>dev</id>
            <activation><activeByDefault>true</activeByDefault></activation>
            <properties>
                <spring.profiles.active>dev</spring.profiles.active>
            </properties>
        </profile>
        <profile>
            <id>prod</id>
            <properties>
                <spring.profiles.active>prod</spring.profiles.active>
            </properties>
        </profile>
    </profiles>

</project>
```

---

## 📁 Estructura de proyecto

```
src/
├── main/
│   ├── java/com/miempresa/api/
│   │   ├── MiApiApplication.java           # Main class
│   │   ├── config/
│   │   │   ├── SecurityConfig.java
│   │   │   ├── CacheConfig.java
│   │   │   ├── AsyncConfig.java
│   │   │   └── OpenApiConfig.java
│   │   ├── controller/
│   │   │   ├── ProductController.java
│   │   │   └── AuthController.java
│   │   ├── service/
│   │   │   ├── ProductService.java
│   │   │   ├── ProductServiceImpl.java
│   │   │   └── AuthService.java
│   │   ├── repository/
│   │   │   ├── ProductRepository.java
│   │   │   └── UserRepository.java
│   │   ├── entity/
│   │   │   ├── Product.java
│   │   │   ├── User.java
│   │   │   └── BaseEntity.java
│   │   ├── dto/
│   │   │   ├── request/
│   │   │   │   ├── CreateProductRequest.java
│   │   │   │   └── LoginRequest.java
│   │   │   └── response/
│   │   │       ├── ProductResponse.java
│   │   │       ├── PageResponse.java
│   │   │       └── ApiResponse.java
│   │   ├── mapper/
│   │   │   └── ProductMapper.java
│   │   ├── security/
│   │   │   ├── JwtAuthFilter.java
│   │   │   ├── JwtService.java
│   │   │   └── UserDetailsServiceImpl.java
│   │   ├── exception/
│   │   │   ├── GlobalExceptionHandler.java
│   │   │   ├── ResourceNotFoundException.java
│   │   │   └── BusinessException.java
│   │   └── util/
│   │       └── PageUtils.java
│   └── resources/
│       ├── application.properties           # Configuración base
│       ├── application-dev.properties       # Sobreescribe para dev
│       ├── application-prod.properties      # Sobreescribe para prod
│       └── db/migration/
│           ├── V1__create_users_table.sql
│           ├── V2__create_products_table.sql
│           └── V3__seed_initial_data.sql
└── test/
    ├── java/com/miempresa/api/
    │   ├── controller/
    │   │   └── ProductControllerTest.java
    │   ├── service/
    │   │   └── ProductServiceTest.java
    │   ├── repository/
    │   │   └── ProductRepositoryTest.java
    │   └── integration/
    │       └── ProductIntegrationTest.java
    └── resources/
        └── application-test.properties
```

---

## ⚙️ application.properties — configuración completa

```properties
# =============================================
# SERVER
# =============================================
server.port=8080
server.servlet.context-path=/api
server.compression.enabled=true
server.compression.mime-types=application/json,application/xml,text/html
server.compression.min-response-size=1024
server.error.include-message=always
server.error.include-binding-errors=always
server.error.include-stacktrace=on_param
server.shutdown=graceful                          # Graceful shutdown
spring.lifecycle.timeout-per-shutdown-phase=30s

# =============================================
# DATASOURCE — PostgreSQL
# =============================================
spring.datasource.url=jdbc:postgresql://localhost:5432/mi_db
spring.datasource.username=postgres
spring.datasource.password=secret
spring.datasource.driver-class-name=org.postgresql.Driver

# HikariCP — connection pool
spring.datasource.hikari.pool-name=HikariPool-Main
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.idle-timeout=600000
spring.datasource.hikari.max-lifetime=1800000
spring.datasource.hikari.auto-commit=true

# =============================================
# JPA / HIBERNATE
# =============================================
spring.jpa.hibernate.ddl-auto=validate           # validate | update | create | create-drop | none
spring.jpa.show-sql=false
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.properties.hibernate.default_batch_fetch_size=20
spring.jpa.properties.hibernate.jdbc.batch_size=25
spring.jpa.properties.hibernate.order_inserts=true
spring.jpa.properties.hibernate.order_updates=true
spring.jpa.open-in-view=false                    # SIEMPRE false en producción

# =============================================
# FLYWAY
# =============================================
spring.flyway.enabled=true
spring.flyway.locations=classpath:db/migration
spring.flyway.baseline-on-migrate=true
spring.flyway.validate-on-migrate=true
spring.flyway.out-of-order=false

# =============================================
# JACKSON — serialización JSON
# =============================================
spring.jackson.serialization.write-dates-as-timestamps=false
spring.jackson.deserialization.fail-on-unknown-properties=false
spring.jackson.default-property-inclusion=non_null
spring.jackson.time-zone=America/Asuncion
spring.jackson.date-format=yyyy-MM-dd'T'HH:mm:ss

# =============================================
# LOGGING
# =============================================
logging.level.root=INFO
logging.level.com.miempresa=DEBUG
logging.level.org.springframework.security=DEBUG
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql=TRACE
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n
logging.pattern.file=%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n
logging.file.name=logs/application.log
logging.file.max-size=10MB
logging.file.max-history=30

# =============================================
# CACHE — Redis
# =============================================
spring.cache.type=redis
spring.cache.redis.time-to-live=3600000          # 1 hora en ms
spring.cache.redis.cache-null-values=false
spring.data.redis.host=localhost
spring.data.redis.port=6379
spring.data.redis.password=
spring.data.redis.timeout=2000ms
spring.data.redis.lettuce.pool.max-active=8
spring.data.redis.lettuce.pool.max-idle=8
spring.data.redis.lettuce.pool.min-idle=2

# =============================================
# MAIL
# =============================================
spring.mail.host=smtp.gmail.com
spring.mail.port=587
spring.mail.username=noreply@miapp.com
spring.mail.password=app-password
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
spring.mail.properties.mail.smtp.connectiontimeout=5000
spring.mail.properties.mail.smtp.timeout=5000

# =============================================
# ACTUATOR
# =============================================
management.endpoints.web.exposure.include=health,info,metrics,prometheus
management.endpoint.health.show-details=when_authorized
management.endpoint.health.show-components=always
management.info.env.enabled=true
management.metrics.export.prometheus.enabled=true
info.app.name=Mi API
info.app.version=1.0.0
info.app.description=API REST con Spring Boot

# =============================================
# ASYNC
# =============================================
spring.task.execution.pool.core-size=5
spring.task.execution.pool.max-size=20
spring.task.execution.pool.queue-capacity=100
spring.task.execution.thread-name-prefix=async-
spring.task.scheduling.pool.size=5
spring.task.scheduling.thread-name-prefix=scheduler-

# =============================================
# MULTIPART — subida de archivos
# =============================================
spring.servlet.multipart.enabled=true
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=10MB

# =============================================
# CUSTOM PROPERTIES — se leen con @ConfigurationProperties
# =============================================
app.jwt.secret=mi-clave-super-secreta-de-256-bits-para-hs256
app.jwt.expiration=86400000                      # 24 horas en ms
app.jwt.refresh-expiration=604800000             # 7 días en ms
app.cors.allowed-origins=http://localhost:3000,https://miapp.com
app.cors.allowed-methods=GET,POST,PUT,PATCH,DELETE,OPTIONS
app.upload.directory=/var/app/uploads
app.upload.max-size=10485760                     # 10 MB en bytes
```

```properties
# application-dev.properties
spring.datasource.url=jdbc:postgresql://localhost:5432/mi_db_dev
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
logging.level.com.miempresa=DEBUG
logging.level.org.springframework.security=DEBUG
spring.devtools.restart.enabled=true
```

```properties
# application-prod.properties
spring.datasource.url=${DATABASE_URL}
spring.datasource.username=${DATABASE_USERNAME}
spring.datasource.password=${DATABASE_PASSWORD}
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false
logging.level.root=WARN
logging.level.com.miempresa=INFO
app.jwt.secret=${JWT_SECRET}
server.error.include-stacktrace=never
server.error.include-message=never
```

---

## 🚀 Main class y runners

```java
@SpringBootApplication
@EnableCaching                    // Activar caché
@EnableAsync                      // Activar @Async
@EnableScheduling                 // Activar @Scheduled
public class MiApiApplication {

    public static void main(String[] args) {
        SpringApplication.run(MiApiApplication.class, args);
    }
}

// CommandLineRunner — ejecutar código al iniciar la app (antes de aceptar tráfico)
@Component
@RequiredArgsConstructor
@Slf4j
public class DataInitializer implements CommandLineRunner {

    private final UserRepository userRepository;

    @Override
    public void run(String... args) {
        if (userRepository.count() == 0) {
            log.info("Inicializando datos de prueba...");
            // cargar datos iniciales
        }
    }
}

// ApplicationRunner — igual pero recibe ApplicationArguments
@Component
@RequiredArgsConstructor
@Slf4j
@Order(1)                         // Orden de ejecución si hay múltiples runners
public class AppStartupRunner implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) {
        log.info("App iniciada. Argumentos: {}", args.getOptionNames());
        if (args.containsOption("init-cache")) {
            // inicializar caché
        }
    }
}
```

---

## 🌐 RestController

```java
@RestController
@RequestMapping("/v1/products")
@RequiredArgsConstructor
@Slf4j
@Tag(name = "Products", description = "CRUD de productos")   // OpenAPI
public class ProductController {

    private final ProductService productService;

    // GET — listar con paginación
    @GetMapping
    public ResponseEntity<PageResponse<ProductResponse>> findAll(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size,
            @RequestParam(defaultValue = "createdAt") String sortBy,
            @RequestParam(defaultValue = "DESC") String sortDir,
            @RequestParam(required = false) String search) {

        var pageable = PageRequest.of(page, size,
            Sort.by(Sort.Direction.fromString(sortDir), sortBy));

        return ResponseEntity.ok(productService.findAll(pageable, search));
    }

    // GET — obtener por ID
    @GetMapping("/{id}")
    public ResponseEntity<ProductResponse> findById(@PathVariable Long id) {
        return ResponseEntity.ok(productService.findById(id));
    }

    // POST — crear
    @PostMapping
    public ResponseEntity<ProductResponse> create(
            @Valid @RequestBody CreateProductRequest request) {
        var product = productService.create(request);
        var location = URI.create("/api/v1/products/" + product.id());
        return ResponseEntity.created(location).body(product);
    }

    // PUT — actualización completa
    @PutMapping("/{id}")
    public ResponseEntity<ProductResponse> update(
            @PathVariable Long id,
            @Valid @RequestBody UpdateProductRequest request) {
        return ResponseEntity.ok(productService.update(id, request));
    }

    // PATCH — actualización parcial
    @PatchMapping("/{id}")
    public ResponseEntity<ProductResponse> patch(
            @PathVariable Long id,
            @RequestBody Map<String, Object> fields) {
        return ResponseEntity.ok(productService.patch(id, fields));
    }

    // DELETE
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        productService.delete(id);
        return ResponseEntity.noContent().build();
    }

    // GET — buscar con múltiples filtros en query params
    @GetMapping("/search")
    public ResponseEntity<List<ProductResponse>> search(
            @RequestParam(required = false) String name,
            @RequestParam(required = false) String category,
            @RequestParam(required = false) BigDecimal minPrice,
            @RequestParam(required = false) BigDecimal maxPrice) {
        return ResponseEntity.ok(productService.search(name, category, minPrice, maxPrice));
    }

    // POST — subir archivo
    @PostMapping("/{id}/image")
    public ResponseEntity<ProductResponse> uploadImage(
            @PathVariable Long id,
            @RequestParam("file") MultipartFile file) {
        return ResponseEntity.ok(productService.uploadImage(id, file));
    }

    // GET — descargar archivo
    @GetMapping("/{id}/export")
    public ResponseEntity<Resource> export(@PathVariable Long id) {
        var resource = productService.exportToCsv(id);
        return ResponseEntity.ok()
            .header(HttpHeaders.CONTENT_DISPOSITION,
                "attachment; filename=\"product-" + id + ".csv\"")
            .contentType(MediaType.TEXT_PLAIN)
            .body(resource);
    }

    // GET — con header personalizado
    @GetMapping("/by-sku/{sku}")
    public ResponseEntity<ProductResponse> findBySku(
            @PathVariable String sku,
            @RequestHeader(value = "X-Tenant-Id", required = false) String tenantId) {
        return ResponseEntity.ok(productService.findBySku(sku, tenantId));
    }
}
```

---

## 📨 ResponseEntity y respuestas estandarizadas

```java
// ApiResponse — wrapper genérico para todas las respuestas
@Getter
@Builder
@JsonInclude(JsonInclude.Include.NON_NULL)
public class ApiResponse<T> {
    private final boolean success;
    private final String message;
    private final T data;
    private final List<String> errors;
    private final LocalDateTime timestamp = LocalDateTime.now();

    public static <T> ApiResponse<T> ok(T data) {
        return ApiResponse.<T>builder().success(true).data(data).build();
    }

    public static <T> ApiResponse<T> ok(T data, String message) {
        return ApiResponse.<T>builder().success(true).data(data).message(message).build();
    }

    public static <T> ApiResponse<T> error(String message) {
        return ApiResponse.<T>builder().success(false).message(message).build();
    }

    public static <T> ApiResponse<T> error(String message, List<String> errors) {
        return ApiResponse.<T>builder().success(false).message(message).errors(errors).build();
    }
}

// PageResponse — respuesta paginada estandarizada
@Getter
@Builder
public class PageResponse<T> {
    private final List<T> content;
    private final int page;
    private final int size;
    private final long totalElements;
    private final int totalPages;
    private final boolean first;
    private final boolean last;
    private final boolean empty;

    public static <T> PageResponse<T> from(Page<T> page) {
        return PageResponse.<T>builder()
            .content(page.getContent())
            .page(page.getNumber())
            .size(page.getSize())
            .totalElements(page.getTotalElements())
            .totalPages(page.getTotalPages())
            .first(page.isFirst())
            .last(page.isLast())
            .empty(page.isEmpty())
            .build();
    }
}

// Respuestas HTTP comunes con ResponseEntity
ResponseEntity.ok(body);                               // 200
ResponseEntity.created(uri).body(body);               // 201
ResponseEntity.accepted().body(body);                 // 202
ResponseEntity.noContent().build();                   // 204
ResponseEntity.badRequest().body(error);              // 400
ResponseEntity.status(HttpStatus.FORBIDDEN).build();  // 403
ResponseEntity.notFound().build();                    // 404
ResponseEntity.status(HttpStatus.CONFLICT).body(err); // 409
ResponseEntity.internalServerError().body(error);     // 500

// Builder detallado
ResponseEntity.status(HttpStatus.CREATED)
    .header("X-Resource-Id", id.toString())
    .contentType(MediaType.APPLICATION_JSON)
    .body(response);
```

---

## ⚠️ Manejo global de excepciones

```java
// Excepciones de dominio
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String resource, Object id) {
        super(String.format("%s no encontrado con id: %s", resource, id));
    }
}

@Getter
public class BusinessException extends RuntimeException {
    private final String code;

    public BusinessException(String code, String message) {
        super(message);
        this.code = code;
    }
}

public class DuplicateResourceException extends RuntimeException {
    public DuplicateResourceException(String message) {
        super(message);
    }
}

// Handler global — captura todas las excepciones de la app
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    // 404 — recurso no encontrado
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ProblemDetail> handleNotFound(
            ResourceNotFoundException ex, HttpServletRequest request) {
        log.warn("Recurso no encontrado: {}", ex.getMessage());
        var problem = ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
        problem.setTitle("Recurso no encontrado");
        problem.setInstance(URI.create(request.getRequestURI()));
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(problem);
    }

    // 400 — error de negocio
    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ProblemDetail> handleBusiness(
            BusinessException ex, HttpServletRequest request) {
        log.warn("Error de negocio [{}]: {}", ex.getCode(), ex.getMessage());
        var problem = ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST, ex.getMessage());
        problem.setTitle("Error de negocio");
        problem.setProperty("code", ex.getCode());
        problem.setInstance(URI.create(request.getRequestURI()));
        return ResponseEntity.badRequest().body(problem);
    }

    // 409 — recurso duplicado
    @ExceptionHandler(DuplicateResourceException.class)
    public ResponseEntity<ProblemDetail> handleDuplicate(
            DuplicateResourceException ex, HttpServletRequest request) {
        var problem = ProblemDetail.forStatusAndDetail(HttpStatus.CONFLICT, ex.getMessage());
        problem.setTitle("Conflicto");
        problem.setInstance(URI.create(request.getRequestURI()));
        return ResponseEntity.status(HttpStatus.CONFLICT).body(problem);
    }

    // 400 — errores de validación (@Valid)
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ProblemDetail> handleValidation(
            MethodArgumentNotValidException ex, HttpServletRequest request) {
        var errors = ex.getBindingResult().getFieldErrors().stream()
            .map(e -> e.getField() + ": " + e.getDefaultMessage())
            .toList();

        var problem = ProblemDetail.forStatus(HttpStatus.BAD_REQUEST);
        problem.setTitle("Error de validación");
        problem.setDetail("La solicitud contiene campos inválidos");
        problem.setProperty("errors", errors);
        problem.setInstance(URI.create(request.getRequestURI()));
        return ResponseEntity.badRequest().body(problem);
    }

    // 400 — errores de constraint en @RequestParam / @PathVariable
    @ExceptionHandler(ConstraintViolationException.class)
    public ResponseEntity<ProblemDetail> handleConstraint(
            ConstraintViolationException ex, HttpServletRequest request) {
        var errors = ex.getConstraintViolations().stream()
            .map(v -> v.getPropertyPath() + ": " + v.getMessage())
            .toList();

        var problem = ProblemDetail.forStatus(HttpStatus.BAD_REQUEST);
        problem.setTitle("Error de validación");
        problem.setProperty("errors", errors);
        problem.setInstance(URI.create(request.getRequestURI()));
        return ResponseEntity.badRequest().body(problem);
    }

    // 400 — JSON malformado
    @ExceptionHandler(HttpMessageNotReadableException.class)
    public ResponseEntity<ProblemDetail> handleMalformedJson(
            HttpMessageNotReadableException ex, HttpServletRequest request) {
        var problem = ProblemDetail.forStatusAndDetail(
            HttpStatus.BAD_REQUEST, "Cuerpo de la solicitud inválido o malformado");
        problem.setTitle("Solicitud inválida");
        return ResponseEntity.badRequest().body(problem);
    }

    // 405 — método HTTP no permitido
    @ExceptionHandler(HttpRequestMethodNotSupportedException.class)
    public ResponseEntity<ProblemDetail> handleMethodNotAllowed(
            HttpRequestMethodNotSupportedException ex) {
        var problem = ProblemDetail.forStatusAndDetail(
            HttpStatus.METHOD_NOT_ALLOWED, ex.getMessage());
        problem.setTitle("Método no permitido");
        return ResponseEntity.status(HttpStatus.METHOD_NOT_ALLOWED).body(problem);
    }

    // 403 — acceso denegado
    @ExceptionHandler(AccessDeniedException.class)
    public ResponseEntity<ProblemDetail> handleAccessDenied(
            AccessDeniedException ex, HttpServletRequest request) {
        var problem = ProblemDetail.forStatusAndDetail(
            HttpStatus.FORBIDDEN, "No tenés permisos para realizar esta acción");
        problem.setTitle("Acceso denegado");
        problem.setInstance(URI.create(request.getRequestURI()));
        return ResponseEntity.status(HttpStatus.FORBIDDEN).body(problem);
    }

    // 500 — error no manejado
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ProblemDetail> handleGeneral(
            Exception ex, HttpServletRequest request) {
        log.error("Error inesperado en {}: {}", request.getRequestURI(), ex.getMessage(), ex);
        var problem = ProblemDetail.forStatusAndDetail(
            HttpStatus.INTERNAL_SERVER_ERROR, "Error interno del servidor");
        problem.setTitle("Error del servidor");
        problem.setInstance(URI.create(request.getRequestURI()));
        return ResponseEntity.internalServerError().body(problem);
    }
}
```

---

## ✅ Validación — Bean Validation

```java
// Request con validaciones
@Getter
@Setter
@NoArgsConstructor
public class CreateProductRequest {

    @NotBlank(message = "El nombre es obligatorio")
    @Size(min = 2, max = 100, message = "El nombre debe tener entre 2 y 100 caracteres")
    private String name;

    @Size(max = 500, message = "La descripción no puede superar los 500 caracteres")
    private String description;

    @NotNull(message = "El precio es obligatorio")
    @DecimalMin(value = "0.01", message = "El precio debe ser mayor a 0")
    @DecimalMax(value = "999999.99", message = "El precio no puede superar 999999.99")
    @Digits(integer = 6, fraction = 2)
    private BigDecimal price;

    @NotNull(message = "El stock es obligatorio")
    @Min(value = 0, message = "El stock no puede ser negativo")
    private Integer stock;

    @NotBlank(message = "La categoría es obligatoria")
    private String category;

    @Pattern(regexp = "^[A-Z]{2,}-\\d{4,}$",
             message = "El SKU debe tener formato XX-NNNN (ej: PROD-0001)")
    private String sku;

    @Email(message = "El email del proveedor es inválido")
    private String supplierEmail;

    @URL(message = "La URL de imagen es inválida")
    private String imageUrl;

    @Valid                        // Validar objeto anidado
    private AddressRequest supplierAddress;

    @NotEmpty(message = "Debe especificar al menos una etiqueta")
    @Size(max = 10, message = "Máximo 10 etiquetas")
    private List<@NotBlank @Size(max = 30) String> tags;

    @Future(message = "La fecha de lanzamiento debe ser futura")
    private LocalDate launchDate;
}

// Validaciones de grupos — ejecutar en orden
@GroupSequence({Default.class, FirstGroup.class, SecondGroup.class})
public interface ValidationOrder {}

// Validator personalizado
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = UniqueSkuValidator.class)
public @interface UniqueSku {
    String message() default "El SKU ya está en uso";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

@Component
@RequiredArgsConstructor
public class UniqueSkuValidator implements ConstraintValidator<UniqueSku, String> {

    private final ProductRepository productRepository;

    @Override
    public boolean isValid(String sku, ConstraintValidatorContext context) {
        if (sku == null || sku.isBlank()) return true; // @NotBlank maneja el null
        return !productRepository.existsBySku(sku);
    }
}

// Validar en @PathVariable y @RequestParam — requiere @Validated en la clase
@RestController
@Validated                        // Activa validación en parámetros del método
@RequestMapping("/v1/products")
public class ProductController {

    @GetMapping("/{id}")
    public ResponseEntity<ProductResponse> findById(
            @PathVariable @Positive(message = "El ID debe ser positivo") Long id) {
        return ResponseEntity.ok(productService.findById(id));
    }

    @GetMapping
    public ResponseEntity<PageResponse<ProductResponse>> findAll(
            @RequestParam @Min(0) int page,
            @RequestParam @Min(1) @Max(100) int size) {
        // ...
    }
}

// Validar manualmente (fuera de un controller)
@Service
@RequiredArgsConstructor
public class ProductService {

    private final Validator validator;

    public void validateManually(Object object) {
        Set<ConstraintViolation<Object>> violations = validator.validate(object);
        if (!violations.isEmpty()) {
            var errors = violations.stream()
                .map(v -> v.getPropertyPath() + ": " + v.getMessage())
                .toList();
            throw new BusinessException("VALIDATION_ERROR",
                "Errores de validación: " + String.join(", ", errors));
        }
    }
}
```

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

---

## 📋 DTOs con record y Lombok

```java
// Java record — inmutable, ideal para DTOs de respuesta (Java 16+)
public record ProductResponse(
    Long id,
    String name,
    String description,
    BigDecimal price,
    Integer stock,
    String category,
    String sku,
    boolean active,
    LocalDateTime createdAt
) {
    // Compact constructor — validaciones
    public ProductResponse {
        Objects.requireNonNull(id, "id no puede ser null");
        Objects.requireNonNull(name, "name no puede ser null");
        if (price != null && price.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("price no puede ser negativo");
        }
    }

    // Métodos de utilidad
    public String formattedPrice() {
        return price != null ? "$" + price.toPlainString() : "Sin precio";
    }

    public boolean isInStock() {
        return stock != null && stock > 0;
    }
}

// Request DTO con Lombok — necesita @Setter para deserialización Jackson
@Getter
@Setter
@NoArgsConstructor
@Builder
public class CreateProductRequest {
    @NotBlank private String name;
    @NotNull @DecimalMin("0.01") private BigDecimal price;
    @NotNull @Min(0) private Integer stock;
    @NotBlank private String category;
}

// DTO de autenticación
public record LoginRequest(
    @NotBlank @Email String email,
    @NotBlank @Size(min = 8) String password
) {}

public record AuthResponse(
    String accessToken,
    String refreshToken,
    long expiresIn,
    UserResponse user
) {}

public record UserResponse(
    Long id,
    String name,
    String email,
    String role,
    LocalDateTime createdAt
) {}

// DTO con datos anidados
public record OrderResponse(
    Long id,
    String status,
    BigDecimal total,
    UserResponse customer,
    List<OrderItemResponse> items,
    AddressResponse shippingAddress,
    LocalDateTime createdAt
) {}

public record OrderItemResponse(
    Long id,
    ProductResponse product,
    Integer quantity,
    BigDecimal unitPrice,
    BigDecimal subtotal
) {}
```

---

## 🗺️ MapStruct — mapeo automático

```java
// Mapper básico
@Mapper(componentModel = "spring")
public interface ProductMapper {

    // Mapeo directo (campos con mismo nombre)
    ProductResponse toResponse(Product product);

    // Mapeo con nombres diferentes
    @Mapping(source = "name", target = "productName")
    @Mapping(source = "category.name", target = "categoryName")  // Acceso anidado
    @Mapping(target = "createdAt", expression = "java(product.getCreatedAt().toString())")
    ProductSummaryResponse toSummary(Product product);

    // Ignorar campos
    @Mapping(target = "id", ignore = true)
    @Mapping(target = "createdAt", ignore = true)
    @Mapping(target = "updatedAt", ignore = true)
    Product toEntity(CreateProductRequest request);

    // Actualizar entidad existente (evita new + set uno por uno)
    @Mapping(target = "id", ignore = true)
    @Mapping(target = "createdAt", ignore = true)
    void updateEntity(UpdateProductRequest request, @MappingTarget Product product);

    // Mapear listas automáticamente
    List<ProductResponse> toResponseList(List<Product> products);

    // Mapear página
    default PageResponse<ProductResponse> toPageResponse(Page<Product> page) {
        return PageResponse.<ProductResponse>builder()
            .content(toResponseList(page.getContent()))
            .page(page.getNumber())
            .size(page.getSize())
            .totalElements(page.getTotalElements())
            .totalPages(page.getTotalPages())
            .first(page.isFirst())
            .last(page.isLast())
            .build();
    }

    // Método de conversión custom
    default String mapStatus(ProductStatus status) {
        return status == null ? null : status.getLabel();
    }
}

// Mapper con dependencia inyectada
@Mapper(componentModel = "spring", uses = {CategoryMapper.class})
public interface ProductMapper {
    // CategoryMapper se usa automáticamente para mapear Category → CategoryResponse
    ProductResponse toResponse(Product product);
}

// Uso en Service
@Service
@RequiredArgsConstructor
public class ProductService {
    private final ProductRepository productRepository;
    private final ProductMapper productMapper;

    public ProductResponse findById(Long id) {
        return productRepository.findById(id)
            .map(productMapper::toResponse)
            .orElseThrow(() -> new ResourceNotFoundException("Product", id));
    }

    public ProductResponse create(CreateProductRequest request) {
        var product = productMapper.toEntity(request);
        product = productRepository.save(product);
        return productMapper.toResponse(product);
    }

    public ProductResponse update(Long id, UpdateProductRequest request) {
        var product = productRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Product", id));
        productMapper.updateEntity(request, product);   // Actualiza in-place
        return productMapper.toResponse(productRepository.save(product));
    }
}
```

---

## 💉 Inyección de dependencias

```java
// Estereotipos
@Component    // Componente genérico
@Service      // Lógica de negocio
@Repository   // Acceso a datos (+ traducción de excepciones de persistencia)
@Controller   // Controlador MVC
@RestController // @Controller + @ResponseBody

// Inyección por constructor — RECOMENDADA (inmutable, testeable)
@Service
@RequiredArgsConstructor    // Lombok genera el constructor con todos los final
public class ProductServiceImpl implements ProductService {
    private final ProductRepository productRepository;
    private final ProductMapper productMapper;
    private final CacheManager cacheManager;

    // NO usar @Autowired en el constructor — Spring lo detecta automáticamente
    // cuando hay un solo constructor
}

// Inyección por campo — no recomendada (dificulta testing)
@Service
public class ProductService {
    @Autowired private ProductRepository productRepository; // ❌ evitar
}

// Inyección por setter — útil para dependencias opcionales
@Service
public class ProductService {
    private NotificationService notificationService;

    @Autowired(required = false)
    public void setNotificationService(NotificationService service) {
        this.notificationService = service;
    }
}

// @Qualifier — múltiples implementaciones del mismo tipo
@Service
@Qualifier("emailNotifier")
public class EmailNotificationService implements NotificationService { ... }

@Service
@Qualifier("smsNotifier")
public class SmsNotificationService implements NotificationService { ... }

@Service
@RequiredArgsConstructor
public class OrderService {
    @Qualifier("emailNotifier")
    private final NotificationService emailNotifier;

    @Qualifier("smsNotifier")
    private final NotificationService smsNotifier;
}

// @Primary — implementación por defecto
@Service
@Primary
public class EmailNotificationService implements NotificationService { ... }

// @Lazy — inicialización diferida
@Service
@Lazy
public class HeavyService { ... }

// @Scope
@Component
@Scope("singleton")     // Default — una instancia por contexto
@Scope("prototype")     // Nueva instancia cada vez que se inyecta
@Scope("request")       // Una instancia por HTTP request
@Scope("session")       // Una instancia por HTTP session
public class MyBean { ... }
```

---

## ⚙️ Configuración — Beans

```java
// @Configuration + @Bean
@Configuration
@RequiredArgsConstructor
public class AppConfig {

    private final AppProperties appProperties;

    // Bean con nombre personalizado
    @Bean("restTemplate")
    public RestTemplate restTemplate() {
        var factory = new SimpleClientHttpRequestFactory();
        factory.setConnectTimeout(5000);
        factory.setReadTimeout(10000);
        return new RestTemplate(factory);
    }

    // Bean condicional
    @Bean
    @ConditionalOnProperty(name = "app.feature.notifications", havingValue = "true")
    public NotificationService notificationService() {
        return new EmailNotificationService();
    }

    // CORS
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        var config = new CorsConfiguration();
        config.setAllowedOrigins(appProperties.getCors().getAllowedOrigins());
        config.setAllowedMethods(List.of("GET","POST","PUT","PATCH","DELETE","OPTIONS"));
        config.setAllowedHeaders(List.of("*"));
        config.setAllowCredentials(true);
        config.setMaxAge(3600L);

        var source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", config);
        return source;
    }

    // ObjectMapper personalizado
    @Bean
    @Primary
    public ObjectMapper objectMapper() {
        var mapper = new ObjectMapper();
        mapper.registerModule(new JavaTimeModule());
        mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
        mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
        mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        return mapper;
    }

    // PasswordEncoder
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);
    }

    // ModelMapper (alternativa a MapStruct)
    @Bean
    public ModelMapper modelMapper() {
        var mapper = new ModelMapper();
        mapper.getConfiguration()
            .setMatchingStrategy(MatchingStrategies.STRICT)
            .setSkipNullEnabled(true);
        return mapper;
    }
}

// AsyncConfig
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        var executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.initialize();
        return executor;
    }

    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return (ex, method, params) ->
            log.error("Error en método async {}: {}", method.getName(), ex.getMessage(), ex);
    }
}
```

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

---

## 🔐 Spring Security

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity(prePostEnabled = true)  // Activa @PreAuthorize, @PostAuthorize
@RequiredArgsConstructor
public class SecurityConfig {

    private final JwtAuthFilter jwtAuthFilter;
    private final UserDetailsService userDetailsService;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(AbstractHttpConfigurer::disable)          // Deshabilitar CSRF (API REST)
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))
            .sessionManagement(session ->
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                // Rutas públicas
                .requestMatchers("/api/v1/auth/**").permitAll()
                .requestMatchers("/api/v1/products", "/api/v1/products/**").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/v1/**").permitAll()
                .requestMatchers("/actuator/health", "/actuator/info").permitAll()
                .requestMatchers("/swagger-ui/**", "/v3/api-docs/**").permitAll()
                // Rutas con rol específico
                .requestMatchers("/api/v1/admin/**").hasRole("ADMIN")
                .requestMatchers(HttpMethod.DELETE, "/api/v1/**").hasRole("ADMIN")
                // Todo lo demás requiere autenticación
                .anyRequest().authenticated())
            // Agregar filtro JWT antes del filtro de autenticación
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)
            // Manejo de errores de seguridad
            .exceptionHandling(ex -> ex
                .authenticationEntryPoint((request, response, authException) -> {
                    response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
                    response.setContentType(MediaType.APPLICATION_JSON_VALUE);
                    response.getWriter().write("""
                        {"error": "No autenticado", "status": 401}
                        """);
                })
                .accessDeniedHandler((request, response, accessDeniedException) -> {
                    response.setStatus(HttpServletResponse.SC_FORBIDDEN);
                    response.setContentType(MediaType.APPLICATION_JSON_VALUE);
                    response.getWriter().write("""
                        {"error": "Acceso denegado", "status": 403}
                        """);
                }))
            .build();
    }

    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }

    @Bean
    public AuthenticationProvider authenticationProvider() {
        var provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(passwordEncoder());
        return provider;
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);
    }
}

// UserDetailsService
@Service
@RequiredArgsConstructor
public class UserDetailsServiceImpl implements UserDetailsService {

    private final UserRepository userRepository;

    @Override
    @Transactional(readOnly = true)
    public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
        return userRepository.findByEmail(email)
            .orElseThrow(() -> new UsernameNotFoundException(
                "Usuario no encontrado: " + email));
    }
}
```

---

## 🔑 JWT — filter y utilidades

```java
// JwtService
@Service
@Slf4j
public class JwtService {

    @Value("${app.jwt.secret}")
    private String jwtSecret;

    @Value("${app.jwt.expiration}")
    private long jwtExpiration;

    @Value("${app.jwt.refresh-expiration}")
    private long refreshExpiration;

    private SecretKey getSigningKey() {
        return Keys.hmacShaKeyFor(Decoders.BASE64.decode(jwtSecret));
    }

    public String generateToken(UserDetails user) {
        return generateToken(Map.of(), user, jwtExpiration);
    }

    public String generateToken(Map<String, Object> extraClaims, UserDetails user) {
        return generateToken(extraClaims, user, jwtExpiration);
    }

    public String generateRefreshToken(UserDetails user) {
        return generateToken(Map.of("type", "refresh"), user, refreshExpiration);
    }

    private String generateToken(Map<String, Object> claims, UserDetails user, long expiration) {
        return Jwts.builder()
            .claims(claims)
            .subject(user.getUsername())
            .issuedAt(new Date())
            .expiration(new Date(System.currentTimeMillis() + expiration))
            .signWith(getSigningKey())
            .compact();
    }

    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }

    public <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {
        return claimsResolver.apply(extractAllClaims(token));
    }

    public boolean isTokenValid(String token, UserDetails userDetails) {
        try {
            final String username = extractUsername(token);
            return username.equals(userDetails.getUsername()) && !isTokenExpired(token);
        } catch (JwtException | IllegalArgumentException e) {
            log.warn("Token JWT inválido: {}", e.getMessage());
            return false;
        }
    }

    private boolean isTokenExpired(String token) {
        return extractClaim(token, Claims::getExpiration).before(new Date());
    }

    private Claims extractAllClaims(String token) {
        return Jwts.parser()
            .verifyWith(getSigningKey())
            .build()
            .parseSignedClaims(token)
            .getPayload();
    }
}

// JwtAuthFilter
@Component
@RequiredArgsConstructor
@Slf4j
public class JwtAuthFilter extends OncePerRequestFilter {

    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
            HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {

        final String authHeader = request.getHeader("Authorization");

        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }

        final String jwt = authHeader.substring(7);

        try {
            final String username = jwtService.extractUsername(jwt);

            if (username != null &&
                    SecurityContextHolder.getContext().getAuthentication() == null) {

                UserDetails userDetails = userDetailsService.loadUserByUsername(username);

                if (jwtService.isTokenValid(jwt, userDetails)) {
                    var authToken = new UsernamePasswordAuthenticationToken(
                        userDetails,
                        null,
                        userDetails.getAuthorities());

                    authToken.setDetails(
                        new WebAuthenticationDetailsSource().buildDetails(request));

                    SecurityContextHolder.getContext().setAuthentication(authToken);
                }
            }
        } catch (Exception e) {
            log.warn("No se pudo autenticar el token JWT: {}", e.getMessage());
        }

        filterChain.doFilter(request, response);
    }
}

// AuthService
@Service
@RequiredArgsConstructor
@Slf4j
public class AuthService {

    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final JwtService jwtService;
    private final AuthenticationManager authenticationManager;

    @Transactional
    public AuthResponse register(RegisterRequest request) {
        if (userRepository.existsByEmail(request.email())) {
            throw new DuplicateResourceException("El email ya está registrado");
        }

        var user = User.builder()
            .name(request.name())
            .email(request.email())
            .password(passwordEncoder.encode(request.password()))
            .role(Role.USER)
            .build();

        userRepository.save(user);
        log.info("Usuario registrado: {}", user.getEmail());

        return buildAuthResponse(user);
    }

    public AuthResponse login(LoginRequest request) {
        authenticationManager.authenticate(
            new UsernamePasswordAuthenticationToken(request.email(), request.password()));

        var user = userRepository.findByEmail(request.email())
            .orElseThrow(() -> new ResourceNotFoundException("User", request.email()));

        log.info("Login exitoso: {}", user.getEmail());
        return buildAuthResponse(user);
    }

    public AuthResponse refresh(String refreshToken) {
        final String email = jwtService.extractUsername(refreshToken);
        var user = userRepository.findByEmail(email)
            .orElseThrow(() -> new ResourceNotFoundException("User", email));

        if (!jwtService.isTokenValid(refreshToken, user)) {
            throw new BusinessException("INVALID_TOKEN", "Refresh token inválido o expirado");
        }

        return buildAuthResponse(user);
    }

    private AuthResponse buildAuthResponse(User user) {
        var claims = Map.<String, Object>of("role", user.getRole().name());
        var accessToken = jwtService.generateToken(claims, user);
        var refreshToken = jwtService.generateRefreshToken(user);

        return new AuthResponse(accessToken, refreshToken,
            jwtService.getJwtExpiration() / 1000, toUserResponse(user));
    }
}

// AuthController
@RestController
@RequestMapping("/v1/auth")
@RequiredArgsConstructor
public class AuthController {

    private final AuthService authService;

    @PostMapping("/register")
    public ResponseEntity<AuthResponse> register(@Valid @RequestBody RegisterRequest request) {
        return ResponseEntity.status(HttpStatus.CREATED).body(authService.register(request));
    }

    @PostMapping("/login")
    public ResponseEntity<AuthResponse> login(@Valid @RequestBody LoginRequest request) {
        return ResponseEntity.ok(authService.login(request));
    }

    @PostMapping("/refresh")
    public ResponseEntity<AuthResponse> refresh(
            @RequestHeader("X-Refresh-Token") String refreshToken) {
        return ResponseEntity.ok(authService.refresh(refreshToken));
    }

    @GetMapping("/me")
    public ResponseEntity<UserResponse> me(@AuthenticationPrincipal UserDetails userDetails) {
        return ResponseEntity.ok(authService.getCurrentUser(userDetails.getUsername()));
    }
}
```

---

## 🔓 Autorización

```java
// Anotaciones de método — requiere @EnableMethodSecurity
@RestController
public class ProductController {

    @GetMapping
    @PreAuthorize("isAuthenticated()")
    public List<ProductResponse> findAll() { ... }

    @PostMapping
    @PreAuthorize("hasRole('ADMIN') or hasRole('MODERATOR')")
    public ProductResponse create(@Valid @RequestBody CreateProductRequest req) { ... }

    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<Void> delete(@PathVariable Long id) { ... }

    // SpEL avanzado — verificar que el usuario sea dueño del recurso
    @PutMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN') or @productSecurity.isOwner(authentication, #id)")
    public ProductResponse update(@PathVariable Long id, @RequestBody UpdateProductRequest req) { ... }

    // @PostAuthorize — verificar DESPUÉS de ejecutar el método
    @GetMapping("/{id}")
    @PostAuthorize("hasRole('ADMIN') or returnObject.ownerId == authentication.name")
    public ProductResponse findById(@PathVariable Long id) { ... }

    // @Secured (legacy, necesita @EnableMethodSecurity(securedEnabled = true))
    @Secured({"ROLE_ADMIN", "ROLE_MODERATOR"})
    public void someMethod() { ... }
}

// Bean de seguridad para lógica compleja
@Component("productSecurity")
@RequiredArgsConstructor
public class ProductSecurityService {

    private final ProductRepository productRepository;

    public boolean isOwner(Authentication authentication, Long productId) {
        return productRepository.findById(productId)
            .map(p -> p.getCreatedBy().equals(authentication.getName()))
            .orElse(false);
    }
}

// Obtener el usuario autenticado en cualquier parte
public User getCurrentUser() {
    return (User) SecurityContextHolder.getContext()
        .getAuthentication().getPrincipal();
}

// @AuthenticationPrincipal — inyectar directamente en el método
@GetMapping("/my-products")
public List<ProductResponse> myProducts(
        @AuthenticationPrincipal User currentUser) {
    return productService.findByUser(currentUser.getId());
}
```

---

## ⚙️ @ConfigurationProperties

```java
// Clase de propiedades tipada
@ConfigurationProperties(prefix = "app")
@Getter
@Setter
@Validated    // Validar propiedades al arrancar
public class AppProperties {

    @NotNull
    private Jwt jwt = new Jwt();

    @NotNull
    private Cors cors = new Cors();

    @NotNull
    private Upload upload = new Upload();

    @Getter @Setter
    public static class Jwt {
        @NotBlank
        private String secret;
        @Positive
        private long expiration = 86400000L;
        @Positive
        private long refreshExpiration = 604800000L;
    }

    @Getter @Setter
    public static class Cors {
        private List<String> allowedOrigins = List.of("*");
        private List<String> allowedMethods = List.of("GET","POST","PUT","DELETE");
    }

    @Getter @Setter
    public static class Upload {
        private String directory = "/tmp/uploads";
        private long maxSize = 10_485_760L;    // 10MB
        private List<String> allowedTypes = List.of("image/jpeg","image/png","application/pdf");
    }
}

// Activar en Main o @Configuration
@SpringBootApplication
@ConfigurationPropertiesScan   // Escanea automáticamente todos los @ConfigurationProperties
public class MiApiApplication { ... }

// O explícito
@EnableConfigurationProperties(AppProperties.class)
@Configuration
public class AppConfig { ... }

// Uso
@Service
@RequiredArgsConstructor
public class JwtService {
    private final AppProperties appProperties;

    public long getExpiration() {
        return appProperties.getJwt().getExpiration();
    }
}
```

---

## 🎭 @Profile y @Conditional

```java
// @Profile — activar beans según el perfil activo
@Configuration
@Profile("dev")
public class DevConfig {
    @Bean
    public DataSource devDataSource() { ... }
}

@Configuration
@Profile("prod")
public class ProdConfig {
    @Bean
    public DataSource prodDataSource() { ... }
}

@Configuration
@Profile("!prod")   // En todos los perfiles excepto prod
public class DevToolsConfig { ... }

@Configuration
@Profile({"dev", "test"})   // En dev O test
public class NonProdConfig { ... }

// En componentes
@Component
@Profile("dev")
public class DevDataInitializer implements CommandLineRunner { ... }

// @Conditional — condiciones programáticas
@Bean
@ConditionalOnProperty(name = "app.feature.redis", havingValue = "true", matchIfMissing = false)
public CacheManager redisCacheManager() { ... }

@Bean
@ConditionalOnMissingBean(CacheManager.class)   // Si no hay otro CacheManager
public CacheManager simpleCacheManager() { ... }

@Bean
@ConditionalOnClass(name = "redis.clients.jedis.Jedis")   // Si la clase existe en classpath
public RedisTemplate<?, ?> redisTemplate() { ... }

@Bean
@ConditionalOnExpression("${app.feature.notifications:false} and '${spring.profiles.active}'=='prod'")
public NotificationService notificationService() { ... }
```

---

## 🌍 Propiedades por ambiente

```bash
# Activar perfil al correr
java -jar mi-api.jar --spring.profiles.active=prod

# Con Maven
mvn spring-boot:run -Dspring-boot.run.profiles=dev

# Variable de entorno
SPRING_PROFILES_ACTIVE=prod java -jar mi-api.jar

# Variables de entorno que sobreescriben properties
# SPRING_DATASOURCE_URL sobreescribe spring.datasource.url
# APP_JWT_SECRET sobreescribe app.jwt.secret
```

---

## 🗄️ Caché

```java
// CacheConfig
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory factory) {
        var config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofHours(1))
            .disableCachingNullValues()
            .serializeKeysWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new GenericJackson2JsonRedisSerializer()));

        // Configuración por caché individual
        var cacheConfigs = Map.of(
            "products", config.entryTtl(Duration.ofMinutes(30)),
            "categories", config.entryTtl(Duration.ofHours(24)),
            "users", config.entryTtl(Duration.ofMinutes(10))
        );

        return RedisCacheManager.builder(factory)
            .cacheDefaults(config)
            .withInitialCacheConfigurations(cacheConfigs)
            .build();
    }
}

// Uso en Service
@Service
@RequiredArgsConstructor
@Slf4j
public class ProductService {

    private final ProductRepository productRepository;

    // @Cacheable — cachear el resultado
    @Cacheable(value = "products", key = "#id",
               condition = "#id > 0",
               unless = "#result == null")
    @Transactional(readOnly = true)
    public ProductResponse findById(Long id) {
        log.debug("Cache miss para product:{}", id);
        return productRepository.findById(id)
            .map(productMapper::toResponse)
            .orElseThrow(() -> new ResourceNotFoundException("Product", id));
    }

    // @CachePut — actualizar caché sin saltear el método
    @CachePut(value = "products", key = "#result.id()")
    @Transactional
    public ProductResponse update(Long id, UpdateProductRequest request) {
        var product = productRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Product", id));
        productMapper.updateEntity(request, product);
        return productMapper.toResponse(productRepository.save(product));
    }

    // @CacheEvict — limpiar caché al modificar datos
    @CacheEvict(value = "products", key = "#id")
    @Transactional
    public void delete(Long id) {
        productRepository.deleteById(id);
    }

    // Limpiar toda la caché de products
    @CacheEvict(value = "products", allEntries = true)
    public void clearProductsCache() {}

    // @Caching — combinar múltiples operaciones de caché
    @Caching(evict = {
        @CacheEvict(value = "products", key = "#id"),
        @CacheEvict(value = "product-list", allEntries = true)
    })
    @Transactional
    public void deactivate(Long id) { ... }
}
```

---

## ⚡ Async

```java
// @Async — ejecutar en thread pool separado
@Service
@RequiredArgsConstructor
@Slf4j
public class NotificationService {

    private final JavaMailSender mailSender;

    // Fire-and-forget — sin esperar resultado
    @Async
    public void sendWelcomeEmail(String email, String name) {
        log.info("Enviando email a: {}", email);
        // ... enviar email en background
    }

    // Retornar CompletableFuture para operaciones async con resultado
    @Async
    public CompletableFuture<Boolean> sendEmailAsync(String email, String subject, String body) {
        try {
            var message = mailSender.createMimeMessage();
            var helper = new MimeMessageHelper(message, true, "UTF-8");
            helper.setTo(email);
            helper.setSubject(subject);
            helper.setText(body, true);
            mailSender.send(message);
            return CompletableFuture.completedFuture(true);
        } catch (Exception e) {
            log.error("Error al enviar email a {}: {}", email, e.getMessage(), e);
            return CompletableFuture.completedFuture(false);
        }
    }
}

// Usar CompletableFuture en Service
@Service
@RequiredArgsConstructor
public class OrderService {

    private final NotificationService notificationService;
    private final InventoryService inventoryService;

    @Transactional
    public OrderResponse create(CreateOrderRequest request) {
        var order = saveOrder(request);

        // Ejecutar en paralelo tras commitear la transacción
        CompletableFuture
            .allOf(
                notificationService.sendEmailAsync(
                    order.getCustomerEmail(), "Pedido confirmado", buildEmailBody(order)),
                inventoryService.reserveStockAsync(order.getItems())
            )
            .exceptionally(ex -> {
                log.error("Error en operaciones post-orden: {}", ex.getMessage());
                return null;
            });

        return orderMapper.toResponse(order);
    }
}
```

---

## ⏱️ Scheduling

```java
@Service
@Slf4j
public class ScheduledTasks {

    // Cada 5 minutos
    @Scheduled(fixedRate = 300_000)
    public void syncInventory() {
        log.info("Sincronizando inventario...");
    }

    // Con delay inicial — 10 segundos después de arrancar, luego cada 60 segundos
    @Scheduled(initialDelay = 10_000, fixedDelay = 60_000)
    public void warmUpCache() {
        log.info("Calentando caché...");
    }

    // Cron — cada día a las 2:00 AM
    @Scheduled(cron = "0 0 2 * * *")
    public void cleanupOldData() {
        log.info("Limpiando datos antiguos...");
    }

    // Cron — lunes a viernes a las 9:00 AM (zona horaria)
    @Scheduled(cron = "0 0 9 * * MON-FRI", zone = "America/Asuncion")
    public void sendDailyReport() {
        log.info("Enviando reporte diario...");
    }

    // Expresiones cron comunes
    // "0 * * * * *"        — cada minuto
    // "0 */5 * * * *"      — cada 5 minutos
    // "0 0 * * * *"        — cada hora
    // "0 0 0 * * *"        — cada día a medianoche
    // "0 0 12 * * *"       — cada día al mediodía
    // "0 0 0 * * MON"      — todos los lunes a medianoche
    // "0 0 0 1 * *"        — primero de cada mes a medianoche
    // "0 0 0 1 1 *"        — 1ro de enero cada año

    // Cron desde properties — permite cambiar sin recompilar
    @Scheduled(cron = "${app.scheduling.cleanup-cron:0 0 2 * * *}")
    public void scheduledCleanup() { ... }
}
```

---

## 📢 Eventos

```java
// Evento de dominio
public record ProductCreatedEvent(Long productId, String sku, String category) {}
public record OrderCompletedEvent(Long orderId, String customerEmail, BigDecimal total) {}

// Publicar eventos
@Service
@RequiredArgsConstructor
public class ProductService {

    private final ApplicationEventPublisher eventPublisher;
    private final ProductRepository productRepository;

    @Transactional
    public ProductResponse create(CreateProductRequest request) {
        var product = productMapper.toEntity(request);
        product = productRepository.save(product);

        // Publicar evento — se procesa DENTRO de la misma transacción
        eventPublisher.publishEvent(
            new ProductCreatedEvent(product.getId(), product.getSku(),
                product.getCategory().getName()));

        return productMapper.toResponse(product);
    }
}

// Escuchar eventos
@Component
@RequiredArgsConstructor
@Slf4j
public class ProductEventListener {

    private final NotificationService notificationService;
    private final SearchIndexService searchIndexService;

    // Listener síncrono — en la misma transacción
    @EventListener
    public void onProductCreated(ProductCreatedEvent event) {
        log.info("Producto creado: {} (SKU: {})", event.productId(), event.sku());
    }

    // Listener asíncrono — en thread separado
    @EventListener
    @Async
    public void indexNewProduct(ProductCreatedEvent event) {
        searchIndexService.index(event.productId());
    }

    // @TransactionalEventListener — se ejecuta DESPUÉS del commit
    // Ideal para side effects que no deben revertirse con la transacción
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    @Async
    public void sendWelcomeEmail(ProductCreatedEvent event) {
        // Si la transacción hizo rollback, esto NO se ejecuta
        notificationService.notifyNewProduct(event.productId());
    }

    @TransactionalEventListener(phase = TransactionPhase.AFTER_ROLLBACK)
    public void handleProductCreationFailed(ProductCreatedEvent event) {
        log.error("Falló la creación del producto: {}", event.sku());
    }
}
```

---

## 📧 Mail

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class MailService {

    private final JavaMailSender mailSender;

    @Value("${spring.mail.username}")
    private String from;

    // Email simple
    public void sendSimple(String to, String subject, String text) {
        var message = new SimpleMailMessage();
        message.setFrom(from);
        message.setTo(to);
        message.setSubject(subject);
        message.setText(text);
        mailSender.send(message);
    }

    // Email HTML con adjuntos
    @Async
    public CompletableFuture<Void> sendHtml(String to, String subject,
            String htmlBody, List<File> attachments) {
        try {
            var message = mailSender.createMimeMessage();
            var helper = new MimeMessageHelper(message, true, "UTF-8");

            helper.setFrom(from);
            helper.setTo(to);
            helper.setSubject(subject);
            helper.setText(htmlBody, true);   // true = es HTML

            for (var file : attachments) {
                helper.addAttachment(file.getName(), file);
            }

            mailSender.send(message);
            log.info("Email enviado a: {}", to);
        } catch (MessagingException e) {
            log.error("Error al enviar email a {}: {}", to, e.getMessage(), e);
        }
        return CompletableFuture.completedFuture(null);
    }

    // Email a múltiples destinatarios
    public void sendToMultiple(List<String> recipients, String subject, String body) {
        var message = new SimpleMailMessage();
        message.setFrom(from);
        message.setTo(recipients.toArray(String[]::new));
        message.setCc("manager@miapp.com");
        message.setSubject(subject);
        message.setText(body);
        mailSender.send(message);
    }
}
```

---

## 📊 Actuator

```java
// Health check personalizado
@Component
public class ExternalApiHealthIndicator implements HealthIndicator {

    private final RestTemplate restTemplate;

    public ExternalApiHealthIndicator(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    @Override
    public Health health() {
        try {
            var response = restTemplate.getForEntity(
                "https://api.externa.com/health", String.class);

            if (response.getStatusCode().is2xxSuccessful()) {
                return Health.up()
                    .withDetail("url", "https://api.externa.com")
                    .withDetail("status", response.getStatusCode())
                    .build();
            }

            return Health.down()
                .withDetail("url", "https://api.externa.com")
                .withDetail("status", response.getStatusCode())
                .build();

        } catch (Exception e) {
            return Health.down()
                .withDetail("error", e.getMessage())
                .build();
        }
    }
}

// Info contributor personalizado
@Component
public class AppInfoContributor implements InfoContributor {

    @Override
    public void contribute(Info.Builder builder) {
        builder.withDetail("api", Map.of(
            "version", "1.0.0",
            "environment", System.getProperty("spring.profiles.active", "default"),
            "buildTime", "2026-01-15T10:30:00Z"
        ));
    }
}

// Métrica personalizada con Micrometer
@Service
@RequiredArgsConstructor
public class ProductService {

    private final MeterRegistry meterRegistry;
    private final Counter productCreatedCounter;

    public ProductService(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
        this.productCreatedCounter = Counter.builder("products.created")
            .description("Total de productos creados")
            .tag("service", "product")
            .register(meterRegistry);
    }

    public ProductResponse create(CreateProductRequest request) {
        var response = doCreate(request);

        productCreatedCounter.increment();

        // Timer — medir duración de operaciones
        Timer.Sample sample = Timer.start(meterRegistry);
        // ... operación ...
        sample.stop(Timer.builder("products.search.duration")
            .description("Duración de búsqueda de productos")
            .register(meterRegistry));

        // Gauge — valor actual (activo en un momento dado)
        Gauge.builder("products.active.count", productRepository, ProductRepository::countByActiveTrue)
            .description("Productos activos")
            .register(meterRegistry);

        return response;
    }
}
```

---

## 🧪 Testing

```java
// Dependencias Maven ya incluidas:
// spring-boot-starter-test incluye: JUnit 5, Mockito, AssertJ, Hamcrest, MockMvc, Testcontainers

// @SpringBootTest — levanta el contexto completo
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Testcontainers
class ProductIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine")
        .withDatabaseName("test_db")
        .withUsername("test")
        .withPassword("test");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private ProductRepository productRepository;

    @BeforeEach
    void setUp() {
        productRepository.deleteAll();
    }

    @Test
    void createProduct_returnsCreated() {
        var request = new CreateProductRequest();
        request.setName("Laptop");
        request.setPrice(new BigDecimal("999.99"));
        request.setStock(10);
        request.setCategory("electronica");

        var response = restTemplate.postForEntity(
            "/api/v1/products", request, ProductResponse.class);

        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(response.getBody()).isNotNull();
        assertThat(response.getBody().name()).isEqualTo("Laptop");
    }
}

// @WebMvcTest — solo la capa web (sin base de datos)
@WebMvcTest(ProductController.class)
@WithMockUser(username = "admin@test.com", roles = "ADMIN")
class ProductControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private ProductService productService;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    void findById_returnsProduct() throws Exception {
        var product = new ProductResponse(1L, "Laptop", "Descripción",
            new BigDecimal("999.99"), 10, "electronica", "PROD-0001",
            true, LocalDateTime.now());

        given(productService.findById(1L)).willReturn(product);

        mockMvc.perform(get("/api/v1/products/1")
                .contentType(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.name").value("Laptop"))
            .andExpect(jsonPath("$.price").value(999.99));
    }

    @Test
    void create_withInvalidRequest_returnsBadRequest() throws Exception {
        var request = new CreateProductRequest();
        // name vacío — debe fallar la validación

        mockMvc.perform(post("/api/v1/products")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.title").value("Error de validación"));
    }

    @Test
    void delete_asUser_returnsForbidden() throws Exception {
        mockMvc.perform(delete("/api/v1/products/1"))
            .andExpect(status().isForbidden());
    }
}

// @DataJpaTest — solo la capa de persistencia (usa H2 por defecto)
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Testcontainers
class ProductRepositoryTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");

    @DynamicPropertySource
    static void props(DynamicPropertyRegistry r) {
        r.add("spring.datasource.url", postgres::getJdbcUrl);
        r.add("spring.datasource.username", postgres::getUsername);
        r.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private ProductRepository productRepository;

    @Autowired
    private TestEntityManager entityManager;

    @Test
    void findBySku_existingSku_returnsProduct() {
        var product = new Product();
        product.setSku("TEST-001");
        product.setName("Test Product");
        product.setPrice(new BigDecimal("50.00"));
        product.setStock(5);
        entityManager.persistAndFlush(product);

        var found = productRepository.findBySku("TEST-001");

        assertThat(found).isPresent();
        assertThat(found.get().getName()).isEqualTo("Test Product");
    }

    @Test
    void existsBySku_nonExistingSku_returnsFalse() {
        assertThat(productRepository.existsBySku("NONEXISTENT")).isFalse();
    }
}

// Test unitario de Service con Mockito
@ExtendWith(MockitoExtension.class)
class ProductServiceTest {

    @Mock
    private ProductRepository productRepository;

    @Mock
    private ProductMapper productMapper;

    @InjectMocks
    private ProductServiceImpl productService;

    @Test
    void findById_existingId_returnsProduct() {
        var product = new Product();
        product.setId(1L);
        product.setName("Laptop");

        var response = new ProductResponse(1L, "Laptop", null,
            BigDecimal.TEN, 5, null, null, true, LocalDateTime.now());

        given(productRepository.findById(1L)).willReturn(Optional.of(product));
        given(productMapper.toResponse(product)).willReturn(response);

        var result = productService.findById(1L);

        assertThat(result.name()).isEqualTo("Laptop");
        verify(productRepository).findById(1L);
        verify(productMapper).toResponse(product);
    }

    @Test
    void findById_nonExistingId_throwsException() {
        given(productRepository.findById(99L)).willReturn(Optional.empty());

        assertThatThrownBy(() -> productService.findById(99L))
            .isInstanceOf(ResourceNotFoundException.class)
            .hasMessageContaining("99");
    }

    @Test
    void create_validRequest_savesProduct() {
        var request = CreateProductRequest.builder()
            .name("Laptop")
            .price(new BigDecimal("999.99"))
            .stock(10)
            .build();

        var product = new Product();
        var savedProduct = new Product();
        savedProduct.setId(1L);

        var response = new ProductResponse(1L, "Laptop", null,
            BigDecimal.TEN, 10, null, null, true, LocalDateTime.now());

        given(productMapper.toEntity(request)).willReturn(product);
        given(productRepository.save(product)).willReturn(savedProduct);
        given(productMapper.toResponse(savedProduct)).willReturn(response);

        var result = productService.create(request);

        assertThat(result.id()).isEqualTo(1L);
        verify(productRepository).save(product);
    }
}
```

---

## 🐳 Docker

```dockerfile
# Dockerfile — multi-stage build
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /build

# Descargar dependencias primero (mejor cache)
COPY mvnw .
COPY .mvn .mvn
COPY pom.xml .
RUN ./mvnw dependency:go-offline -q

# Compilar
COPY src ./src
RUN ./mvnw package -DskipTests -q

# Extraer layers para cache eficiente entre deploys
FROM eclipse-temurin:21-jdk-alpine AS extractor
WORKDIR /build
COPY --from=builder /build/target/*.jar app.jar
RUN java -Djarmode=layertools -jar app.jar extract

# Imagen final — JRE (más pequeño que JDK)
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app

# Usuario sin privilegios
RUN addgroup -S spring && adduser -S spring -G spring

# Copiar layers en orden de menor a mayor frecuencia de cambio
COPY --from=extractor /build/dependencies .
COPY --from=extractor /build/spring-boot-loader .
COPY --from=extractor /build/snapshot-dependencies .
COPY --from=extractor /build/application .

USER spring

# Variables de entorno con valores por defecto
ENV JAVA_OPTS="-Xms256m -Xmx512m -XX:+UseG1GC"
ENV SPRING_PROFILES_ACTIVE=prod

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD wget -qO- http://localhost:8080/api/actuator/health || exit 1

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS org.springframework.boot.loader.launch.JarLauncher"]
```

```yaml
# docker-compose.yml — entorno de desarrollo
services:
  api:
    build: .
    container_name: mi-api
    ports:
      - "8080:8080"
    environment:
      SPRING_PROFILES_ACTIVE: dev
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/mi_db
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: postgres
      SPRING_DATA_REDIS_HOST: redis
      APP_JWT_SECRET: dev-secret-key-only-for-development-256bits
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    volumes:
      - ./logs:/app/logs
    restart: unless-stopped
    networks:
      - backend

  db:
    image: postgres:16-alpine
    container_name: mi-db
    environment:
      POSTGRES_DB: mi_db
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d mi_db"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend

  redis:
    image: redis:7-alpine
    container_name: mi-redis
    command: redis-server --save 60 1 --loglevel warning
    ports:
      - "6379:6379"
    volumes:
      - redisdata:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3
    networks:
      - backend

  mailhog:
    image: mailhog/mailhog
    container_name: mi-mailhog
    ports:
      - "1025:1025"   # SMTP
      - "8025:8025"   # Web UI
    networks:
      - backend

volumes:
  pgdata:
  redisdata:

networks:
  backend:
    driver: bridge
```

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

### Comandos Maven esenciales

```bash
# Compilar
mvn compile

# Ejecutar tests unitarios
mvn test

# Ejecutar tests de integración
mvn verify

# Ejecutar tests + build
mvn clean install

# Omitir tests
mvn clean install -DskipTests

# Correr la app
mvn spring-boot:run

# Correr con perfil específico
mvn spring-boot:run -Dspring-boot.run.profiles=dev

# Generar JAR
mvn clean package

# Ver árbol de dependencias
mvn dependency:tree

# Ver dependencias desactualizadas
mvn versions:display-dependency-updates

# Actualizar versión del proyecto
mvn versions:set -DnewVersion=1.1.0

# Ver plugins configurados
mvn help:describe -Dplugin=spring-boot
```
