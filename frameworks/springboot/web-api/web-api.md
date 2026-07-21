# Spring Boot — Web / API REST

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

