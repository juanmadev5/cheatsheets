# Android Jetpack Compose — Testing

---

## 🧪 Testing

```kotlin
// Unit test — ViewModel
class ProductsViewModelTest {
    private val getProductsUseCase: GetProductsUseCase = mockk()
    private lateinit var viewModel: ProductsViewModel

    @Before
    fun setup() {
        viewModel = ProductsViewModel(getProductsUseCase)
    }

    @Test
    fun `loadProducts emite Success cuando el use case retorna datos`() = runTest {
        val products = listOf(Product(id = 1, name = "Test"))
        coEvery { getProductsUseCase() } returns Result.success(products)

        viewModel.loadProducts()

        val state = viewModel.uiState.value
        assertIs<UiState.Success>(state)
        assertEquals(products, state.products)
    }
}

// Composable UI test
class ProductsScreenTest {
    @get:Rule
    val composeTestRule = createComposeRule()

    @Test
    fun `muestra lista cuando el estado es Success`() {
        val products = listOf(Product(id = 1, name = "Laptop"))

        composeTestRule.setContent {
            MyAppTheme {
                ProductsScreen(
                    uiState = UiState.Success(products),
                    onDelete = {},
                )
            }
        }

        composeTestRule.onNodeWithText("Laptop").assertIsDisplayed()
        composeTestRule.onNodeWithTag("product_list").assertIsDisplayed()
    }

    @Test
    fun `muestra loading indicator cuando carga`() {
        composeTestRule.setContent {
            MyAppTheme { ProductsScreen(uiState = UiState.Loading, onDelete = {}) }
        }
        composeTestRule.onNodeWithContentDescription("Cargando").assertIsDisplayed()
    }
}
```

