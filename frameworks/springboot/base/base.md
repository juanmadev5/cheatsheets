# Spring Boot — Base

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
        <version>4.1.0</version>
        <relativePath/>
    </parent>

    <groupId>com.miempresa</groupId>
    <artifactId>mi-api</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
    <name>mi-api</name>
    <description>API REST con Spring Boot</description>

    <properties>
        <java.version>25</java.version>
        <mapstruct.version>1.6.3</mapstruct.version>
        <lombok.version>1.18.46</lombok.version>
        <testcontainers.version>2.0.5</testcontainers.version>
        <jjwt.version>0.13.0</jjwt.version>
    </properties>

    <dependencies>

        <!-- ===== WEB ===== -->
        <!-- spring-boot-starter-web fue renombrado a spring-boot-starter-webmvc en Spring Boot 4 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webmvc</artifactId>
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
        <!-- Desde Spring Boot 4, flyway-core ya no se declara suelto: se usa el starter -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-flyway</artifactId>
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
        <!-- spring-security-test ahora se resuelve vía el starter de testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!-- @SpringBootTest ya no autoconfigura TestRestTemplate — requiere estas dos dependencias -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-resttestclient</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-restclient</artifactId>
            <scope>test</scope>
        </dependency>

        <!-- H2 para tests en memoria -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>test</scope>
        </dependency>

        <!-- Testcontainers — módulos renombrados con prefijo testcontainers- desde la 2.0 -->
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>testcontainers-junit-jupiter</artifactId>
            <version>${testcontainers.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>testcontainers-postgresql</artifactId>
            <version>${testcontainers.version}</version>
            <scope>test</scope>
        </dependency>

        <!-- Habilita @ServiceConnection para conectar Testcontainers con la app automáticamente -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-testcontainers</artifactId>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <build>
        <plugins>

            <!-- Spring Boot Maven Plugin -->
            <!-- Lombok ya está marcado <optional>true</optional> arriba, y desde el plugin 3.5.7
                 las dependencias optional se excluyen del JAR final por defecto — no hace falta configurarlo -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
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


## ⚙️ Comandos Maven esenciales

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

