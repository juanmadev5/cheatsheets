# Spring Boot — Features

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

