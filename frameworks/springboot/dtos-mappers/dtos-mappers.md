# Spring Boot — DTOs y Mappers

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

