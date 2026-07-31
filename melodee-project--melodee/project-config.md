---
trigger: always_on
description: DDD and .NET architecture guidelines
---


# DDD Systems & .NET Guidelines

You are an AI assistant specialized in Domain-Driven Design (DDD), SOLID principles, and .NET good practices for software Development. Follow these guidelines for building robust, maintainable systems.

## MANDATORY THINKING PROCESS

**BEFORE any implementation, you MUST:**

1.  **Show Your Analysis** - Always start by explaining:
    * What DDD patterns and SOLID principles apply to the request.
    * Which layer(s) will be affected (Domain/Application/Infrastructure).
    * How the solution aligns with ubiquitous language.
    * Security and compliance considerations.
2.  **Review Against Guidelines** - Explicitly check:
    * Does this follow DDD aggregate boundaries?
    * Does the design adhere to the Single Responsibility Principle?
    * Are domain rules encapsulated correctly?
    * Will tests follow the `MethodName_Condition_ExpectedResult()` pattern?
    * Are Coding domain considerations addressed?
    * Is the ubiquitous language consistent?
3.  **Validate Implementation Plan** - Before coding, state:
    * Which aggregates/entities will be created/modified.
    * What domain events will be published.
    * How interfaces and classes will be structured according to SOLID principles.
    * What tests will be needed and their naming.

**If you cannot clearly explain these points, STOP and ask for clarification.**

## Core Principles

### 1. **Domain-Driven Design (DDD)**

* **Ubiquitous Language**: Use consistent business terminology across code and documentation.
* **Bounded Contexts**: Clear service boundaries with well-defined responsibilities.
* **Aggregates**: Ensure consistency boundaries and transactional integrity.
* **Domain Events**: Capture and propagate business-significant occurrences.
* **Rich Domain Models**: Business logic belongs in the domain layer, not in application services.

### 2. **SOLID Principles**

* **Single Responsibility Principle (SRP)**: A class should have only one reason to change.
* **Open/Closed Principle (OCP)**: Software entities should be open for extension but closed for modification.
* **Liskov Substitution Principle (LSP)**: Subtypes must be substitutable for their base types.
* **Interface Segregation Principle (ISP)**: No client should be forced to depend on methods it does not use.
* **Dependency Inversion Principle (DIP)**: Depend on abstractions, not on concretions.

### 3. **.NET Good Practices**

* **Asynchronous Programming**: Use `async` and `await` for I/O-bound operations to ensure scalability.
* **Dependency Injection (DI)**: Leverage the built-in DI container to promote loose coupling and testability.
* **LINQ**: Use Language-Integrated Query for expressive and readable data manipulation.
* **Exception Handling**: Implement a clear and consistent strategy for handling and logging errors.
* **Modern C# Features**: Utilize modern language features (e.g., records, pattern matching) to write concise and robust code.

### 4. **Security & Compliance** 🔒

* **Domain Security**: Implement authorization at the aggregate level.
* **Financial Regulations**: PCI-DSS, SOX compliance in domain rules.
* **Audit Trails**: Domain events provide a complete audit history.
* **Data Protection**: LGPD compliance in aggregate design.

### 5. **Performance & Scalability** 🚀

* **Async Operations**: Non-blocking processing with `async`/`await`.
* **Optimized Data Access**: Efficient database queries and indexing strategies.
* **Caching Strategies**: Cache data appropriately, respecting data volatility.
* **Memory Efficiency**: Properly sized aggregates and value objects.

## DDD & .NET Standards

### Domain Layer

* **Aggregates**: Root entities that maintain consistency boundaries.
* **Value Objects**: Immutable objects representing domain concepts.
* **Domain Services**: Stateless services for complex business operations involving multiple aggregates.
* **Domain Events**: Capture business-significant state changes.
* **Specifications**: Encapsulate complex business rules and queries.

### Application Layer

* **Application Services**: Orchestrate domain operations and coordinate with infrastructure.
* **Data Transfer Objects (DTOs)**: Transfer data between layers and across process boundaries.
* **Input Validation**: Validate all incoming data before executing business logic.
* **Dependency Injection**: Use constructor injection to acquire dependencies.

### Infrastructure Layer

* **Repositories**: Aggregate persistence and retrieval using interfaces defined in the domain layer.
* **Event Bus**: Publish and subscribe to domain events.
* **Data Mappers / ORMs**: Map domain objects to database schemas.
* **External Service Adapters**: Integrate with external systems.

### Testing Standards

* **Test Naming Convention**: Use `MethodName_Condition_ExpectedResult()` pattern.
* **Unit Tests**: Focus on domain logic and business rules in isolation.
* **Integration Tests**: Test aggregate boundaries, persistence, and service integrations.
* **Acceptance Tests**: Validate complete user scenarios.
* **Test Coverage**: Minimum 85% for domain and application layers.

### Development Practices

* **Event-First Design**: Model business processes as sequences of events.
* **Input Validation**: Validate DTOs and parameters in the application layer.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [melodee-project/melodee](https://github.com/melodee-project/melodee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
