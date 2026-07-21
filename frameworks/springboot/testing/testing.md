# Spring Boot — Testing

---

## 🧪 Testing

```java
// Dependencias Maven ya incluidas:
// spring-boot-starter-test incluye: JUnit 5, Mockito, AssertJ, Hamcrest, MockMvc
// spring-boot-starter-security-test agrega soporte de testing de Spring Security (@WithMockUser)
// spring-boot-testcontainers habilita @ServiceConnection
// spring-boot-resttestclient + spring-boot-restclient habilitan TestRestTemplate
// Testcontainers (testcontainers-junit-jupiter, testcontainers-postgresql) se agrega aparte

// @SpringBootTest — levanta el contexto completo
// @ServiceConnection conecta el container con la app automáticamente (reemplaza a @DynamicPropertySource)
// @AutoConfigureTestRestTemplate es obligatorio: @SpringBootTest ya no autoconfigura TestRestTemplate
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureTestRestTemplate
@Testcontainers
class ProductIntegrationTest {

    @Container
    @ServiceConnection
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine")
        .withDatabaseName("test_db")
        .withUsername("test")
        .withPassword("test");

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

    // @MockBean está deprecado — @MockitoBean es el reemplazo (Spring Framework 7)
    @MockitoBean
    private ProductService productService;

    // Jackson 3: el bean autoconfigurado es JsonMapper (inmutable), no ObjectMapper
    @Autowired
    private JsonMapper jsonMapper;

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
                .content(jsonMapper.writeValueAsString(request)))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.title").value("Error de validación"));
    }

    @Test
    void delete_asUser_returnsForbidden() throws Exception {
        mockMvc.perform(delete("/api/v1/products/1"))
            .andExpect(status().isForbidden());
    }
}

// @DataJpaTest — solo la capa de persistencia
// replace = NONE evita que Spring sustituya el datasource por un H2 embebido
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Testcontainers
class ProductRepositoryTest {

    @Container
    @ServiceConnection
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");

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

