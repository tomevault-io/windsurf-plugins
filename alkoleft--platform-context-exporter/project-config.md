---
trigger: always_on
description: Java Integration testing guidelines
---

# Java Integration testing guidelines

These guidelines aim to ensure consistency, reliability, and maintainability of integration tests within the project.

## Table of contents

- Rule 1: Define Clear Scope and Purpose for Integration Tests
- Rule 2: Manage Test Environment & Dependencies with Testcontainers
- Rule 3: Utilize RestAssured for Robust API Testing
- Rule 4: Implement Consistent Data Management Strategies
- Rule 5: Maintain Clear Test Structure and Assertions
- Rule 6: Optimize for Performance and Ensure Proper Cleanup

## Rule 1: Define Clear Scope and Purpose for Integration Tests

Title: Clearly Define the Scope and Purpose of Each Integration Test
Description:
- Integration tests must verify the interaction between multiple components or systems (e.g., service layer with database, service-to-service communication over HTTP).
- Clearly define the boundary of each integration test. What specific interaction, contract, or flow is being tested?
- Prefer integration tests for verifying contracts between services (APIs) and interactions with external dependencies (databases, message queues, etc.).
- Avoid replicating complex business logic in integration tests if it is already thoroughly covered by unit tests. Focus on the integration points.

**Good example:**
```java
// Assume: ProductService interacts with ProductRepository (database) and NotificationService (external HTTP)

// @SpringBootTest // or similar context for the test
// @Testcontainers // if using Testcontainers
public class ProductServiceIT {

    // @Autowired
    // private ProductService productService;
    
    // @Autowired
    // private ProductRepository productRepository; // To verify DB state

    // Mock or use a Testcontainer for NotificationService if its actual calls are out of scope
    // @MockBean
    // private NotificationService mockNotificationService;

    // @Test
    void should_createProduct_saveToDatabase_and_sendNotification() {
        // Scope: Test the flow of creating a product, ensuring it's saved,
        // and that a notification attempt is made.

        // Given: A product DTO
        // ProductDto newProductDto = new ProductDto("Laptop X1", 1500.00);

        // When: ProductService creates the product
        // Product createdProduct = productService.createProduct(newProductDto);

        // Then: Verify interactions
        // 1. Product is saved in the database (verify via repository or direct query)
        // Optional<ProductEntity> savedEntity = productRepository.findById(createdProduct.getId());
        // assertThat(savedEntity).isPresent();
        // assertThat(savedEntity.get().getName()).isEqualTo("Laptop X1");

        // 2. Notification service was called (verify via mock or wiremock if testing HTTP contract)
        // verify(mockNotificationService).sendProductCreationNotification(any(Product.class));
        System.out.println("Conceptual test: Product creation flow verified.");
    }
}
```

**Bad Example:**
```java
// @SpringBootTest
public class OverlappingProductLogicIT {

    // @Autowired
    // private ProductService productService;

    // @Test
    void should_calculateComplexPricing_duringProductCreation() {
        // Bad: This test might be re-testing complex pricing logic
        // that should already be unit-tested in ProductService or a PricingEngine unit test.
        // The integration test should focus on whether ProductService correctly integrates
        // with the database and other services during creation, assuming pricing logic is correct.
        
        // ProductDto productWithComplexPricing = new ProductDto("ComplexItem", 10.0, List.of(new DiscountRule(...)));
        // Product createdProduct = productService.createProduct(productWithComplexPricing);
        
        // If asserts here are deeply checking specific price calculations, it's likely a unit test concern.
        // assertThat(createdProduct.getFinalPrice()).isEqualTo(9.99); // This might be too specific for an IT
        System.out.println("Conceptual bad test: Replicating unit test logic for pricing.");
    }
}
```

## Rule 2: Manage Test Environment & Dependencies with Testcontainers

Title: Use Testcontainers for Reliable Management of External Dependencies
Description:
- Use Testcontainers (`org.testcontainers:testcontainers`) to manage external dependencies (databases, message brokers, caches, other services) required for the test. Avoid relying on pre-existing, shared external environments to ensure test isolation and reproducibility.
- Declare containerized dependencies using `@Testcontainers` and `@Container` annotations for JUnit 5 integration (`org.testcontainers:junit-jupiter`). Manage container lifecycles appropriately (per test suite using `static @Container` or per test method, favoring suite-level for performance).
- Use official or well-maintained Docker images for dependencies. Pin image versions (e.g., `"postgres:15-alpine"`) to ensure reproducible builds.
- Configure containers programmatically (ports, environment variables, wait strategies) within the test setup. Use `Wait.for...` strategies (e.g., `Wait.forHttp("/health")`, `Wait.forLogMessage(...)`) to ensure containers are ready before tests run.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alkoleft/platform-context-exporter](https://github.com/alkoleft/platform-context-exporter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
