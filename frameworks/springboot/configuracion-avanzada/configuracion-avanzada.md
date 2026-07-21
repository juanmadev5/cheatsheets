# Spring Boot — Configuración avanzada

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

