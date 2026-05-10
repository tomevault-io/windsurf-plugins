---
trigger: always_on
description: Type Design Thinking in Java
---

# Type Design Thinking in Java

Type design thinking in Java applies typography principles to code structure and organization. Just as typography creates readable, accessible text, thoughtful type design in Java produces maintainable, comprehensible code.

## Implementing These Principles

1.  **Start with domain modeling**: Sketch your type system before coding.
2.  **Create a type style guide**: Document naming conventions and patterns.
3.  **Review for type consistency**: Periodically check for style adherence.
4.  **Refactor toward clearer type expressions**: Improve existing code.
5.  **Use tools to enforce style**: Configure linters and static analyzers.

Remember, good type design in Java is about communication - making your code's intent clear both to the compiler and to other developers.

## Table of contents

- Rule 1: Establish a Clear Type Hierarchy
- Rule 2: Use Consistent Naming Conventions (Your Type's "Font Family")
- Rule 3: Embrace Whitespace (Kerning and Leading)
- Rule 4: Create Type-Safe Wrappers (Type as Communication)
- Rule 5: Leverage Generic Type Parameters (Responsive Typography)
- Rule 6: Create Domain-Specific Languages (Typography with Character)
- Rule 7: Use Consistent Type "Weights" (Bold, Regular, Light)
- Rule 8: Apply Type Contrast Through Interfaces
- Rule 9: Create Type Alignment Through Method Signatures
- Rule 10: Design for Clear Type Readability and Comprehension
- Rule 11: Use BigDecimal for Precision-Sensitive Calculations
- Rule 12: Strategic Type Selection for Methods and Algorithms

## Rule 1: Establish a Clear Type Hierarchy

Title: Establish a Clear Type Hierarchy
Description: This rule focuses on organizing classes and interfaces into a logical structure using inheritance and composition. A clear hierarchy makes the relationships between types explicit, improving code navigation and understanding. It often involves using nested static classes for closely related types.

**Good example:**

```java
// GOOD: Clear type hierarchy with descriptive names
public class OrderManagement {
    public static class Order {
        private List<OrderItem> items;
        private Customer customer;
        // ...
    }

    public static class OrderItem {
        private Product product;
        private int quantity;
        // ...
    }
}
```

**Bad Example:**

```java
// AVOID: Flat structure with ambiguous names
public class Order {
    private List<Item> items;
    private User user;
    // ...
}
```

## Rule 2: Use Consistent Naming Conventions (Your Type's "Font Family")

Title: Use Consistent Naming Conventions (Your Type's "Font Family")
Description: This rule emphasizes using uniform patterns for naming classes, interfaces, methods, and variables. Consistency in naming acts like a consistent font family in typography, making the code easier to read, predict, and maintain across the entire project.

**Good example:**

```java
// GOOD: Consistent naming patterns
interface PaymentProcessor { void process(Payment payment); }
interface ShippingCalculator { BigDecimal calculate(Order order); }
interface TaxProvider { Tax getTaxFor(Address address); }
```

**Bad Example:**

```java
// AVOID: Inconsistent naming patterns
interface PaymentProcessor { void handlePayment(Payment p); }
interface ShipCalc { BigDecimal getShippingCost(Order o); }
interface TaxSystem { Tax lookupTaxRate(Address addr); }
```

## Rule 3: Embrace Whitespace (Kerning and Leading)

Title: Embrace Whitespace (Kerning and Leading)
Description: This rule advocates for the strategic use of blank lines and spacing within code, analogous to kerning and leading in typography. Proper whitespace improves readability by visually separating logical blocks of code, making it easier to scan and comprehend.

**Good example:**

```java
// GOOD: Proper spacing for readability
public Order processOrder(Cart cart, Customer customer) {
    // Validate inputs
    validateCart(cart);
    validateCustomer(customer);

    // Create order
    Order order = new Order(customer);
    cart.getItems().forEach(item ->
        order.addItem(item.getProduct(), item.getQuantity())
    );

    // Calculate totals
    order.calculateSubtotal();
    order.calculateTax();

    return order;
}
```

**Bad Example:**

```java
// AVOID: Dense, difficult to parse code
public Order processOrder(Cart cart,Customer customer){
    validateCart(cart);validateCustomer(customer);
    Order order=new Order(customer);
    cart.getItems().forEach(item->order.addItem(item.getProduct(),item.getQuantity()));
    order.calculateSubtotal();order.calculateTax();
    return order;
}
```

## Rule 4: Create Type-Safe Wrappers (Type as Communication)

Title: Create Type-Safe Wrappers (Type as Communication)
Description: This rule encourages wrapping primitive types or general-purpose types (like String) in domain-specific types. These wrapper types enhance type safety by enforcing invariants at compile-time and clearly communicate the intended meaning and constraints of data.

**Good example:**

```java
// GOOD: Type-safe wrappers communicate intent
public class EmailAddress {
    private final String value;

    public EmailAddress(String email) {
        if (!isValid(email)) {
            throw new IllegalArgumentException("Invalid email format");
        }

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alkoleft/platform-context-exporter](https://github.com/alkoleft/platform-context-exporter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
