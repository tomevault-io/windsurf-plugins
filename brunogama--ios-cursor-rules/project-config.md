---
trigger: always_on
description: This guide outlines Swift testing principles and practices, focusing on creating effective unit tests for Swift code.
---

# Swift Testing Guide

This guide outlines Swift testing principles and practices, focusing on creating effective unit tests for Swift code.

<rule>
name: swift_testing
filters:
  - type: file_change
    pattern: "*Tests.swift"
  - type: command
    pattern: "@swift-tests"

actions:
  - type: suggest
    message: |
      ## Swift Testing Guidelines
      
      In this project, every public function must have unit tests. We follow these guidelines:

      ### Test Structure
      
      #### Naming Convention
      
      - Test class name: `[ClassUnderTest]Tests`
      - Test method name: `test_[FunctionUnderTest]_[Scenario]_[ExpectedResult]`
      
      **Example:**
      ```swift
      class UserAuthenticationTests: XCTestCase {
          func test_authenticate_withValidCredentials_shouldReturnToken() { ... }
          func test_authenticate_withInvalidPassword_shouldThrowAuthError() { ... }
      }
      ```
      
      #### Arrangement
      
      Every test should follow the Arrange-Act-Assert (AAA) pattern:
      
      ```swift
      func test_addItem_withValidProduct_shouldAddToCart() {
          // Arrange
          let cart = ShoppingCart(id: CartID())
          let product = Product(id: ProductID(), name: "Test Product", price: Money(amount: 10, currency: .usd))
          
          // Act
          try! cart.addItem(product: product, quantity: 1)
          
          // Assert
          XCTAssertEqual(cart.items.count, 1)
          XCTAssertEqual(cart.total()?.amount, 10)
      }
      ```
      
      ### Types of Tests
      
      #### 1. Unit Tests
      
      - Tests a single unit of code in isolation
      - Uses mocks/stubs for dependencies
      - Fast and reliable
      - Located in target's matching test target (e.g., DomainTests for Domain code)
      
      ```swift
      func test_placeOrder_withValidItems_shouldCreateOrderSuccessfully() {
          // Arrange
          let mockOrderRepository = MockOrderRepository()
          let mockProductRepository = MockProductRepository()
          let orderService = OrderService(
              orderRepository: mockOrderRepository,
              productRepository: mockProductRepository
          )
          
          let items = [OrderItem(productID: ProductID(), quantity: 1, price: Money(amount: 10, currency: .usd))]
          
          // Act
          let result = orderService.placeOrder(items: items, customerID: CustomerID())
          
          // Assert
          XCTAssertTrue(result.isSuccess)
          XCTAssertEqual(mockOrderRepository.savedOrders.count, 1)
      }
      ```
      
      #### 2. Integration Tests
      
      - Tests interaction between multiple components
      - Typically involves real implementations rather than mocks
      - Located in separate test targets
      
      ```swift
      func test_orderFlow_endToEnd_shouldProcessOrderSuccessfully() {
          // Tests the full order flow from cart to confirmation
          // Uses real implementations for most components
      }
      ```
      
      ### Mocking Guidelines
      
      - Create mocks for protocols, not for concrete classes
      - Use protocol-based dependencies to make testing easier
      - Name mocks clearly with the `Mock` prefix
      
      ```swift
      protocol OrderRepository {
          func save(_ order: Order) throws
          func findByID(_ id: OrderID) -> Order?
      }
      
      class MockOrderRepository: OrderRepository {
          var savedOrders: [Order] = []
          var ordersToReturn: [OrderID: Order] = [:]
          
          func save(_ order: Order) throws {
              savedOrders.append(order)
          }
          
          func findByID(_ id: OrderID) -> Order? {
              return ordersToReturn[id]
          }
      }
      ```
      
      ### Testing Value Objects
      
      Test both valid and invalid initialization:
      
      ```swift
      func test_emailInitialization_withValidEmail_shouldSucceed() {
          // Arrange & Act
          let email = Email(value: "user@example.com")
          
          // Assert
          XCTAssertNotNil(email)
          XCTAssertEqual(email?.value, "user@example.com")
      }
      
      func test_emailInitialization_withInvalidEmail_shouldReturnNil() {
          // Arrange & Act
          let email = Email(value: "invalid-email")
          
          // Assert
          XCTAssertNil(email)
      }
      ```
      
      ### Testing Entities & Aggregates
      
      Test business rules and invariants:
      
      ```swift
      func test_order_cannotBeFinalized_whenEmpty() {
          // Arrange
          let order = Order(id: OrderID(), customerID: CustomerID())
          
          // Act
          let result = order.finalize()
          
          // Assert
          XCTAssertEqual(result, .failure(.emptyOrder))
      }
      ```
      
      ### Testing Use Cases
      
      Test input/output behavior and interactions:
      
      ```swift
      func test_placeOrderUseCase_shouldCoordinateRepositories() {
          // Arrange
          let mockOrderRepository = MockOrderRepository()
          let mockNotificationService = MockNotificationService()
          

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brunogama/ios-cursor-rules](https://github.com/brunogama/ios-cursor-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
