# Spring Boot — Inyección de dependencias

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

    // Jackson 3 — el JsonMapper es inmutable, no existe un builder tipo Jackson2ObjectMapperBuilder
    // La mayoría de la configuración va por spring.jackson.* en application.properties;
    // para control programático se registra un JsonMapperBuilderCustomizer
    @Bean
    public JsonMapperBuilderCustomizer jsonMapperCustomizer() {
        return builder -> builder.enable(SerializationFeature.INDENT_OUTPUT);
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

