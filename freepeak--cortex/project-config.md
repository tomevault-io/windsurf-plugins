---
trigger: always_on
description: 1. **Clean Architecture**
---

# IDE Cursor Rules for Golang Development

## Architecture & Structure

1. **Clean Architecture**
   - Maintain clear separation between layers: domain, repository, usecase, handler
   - Follow dependency rule: inner layers must not depend on outer layers
   - Domain entities should not have dependencies on frameworks or external libraries
   - Use dependency injection for infrastructure dependencies

2. **Project Structure**
   ```
root/
├── cmd
│   ├── app
│   └── tools
├── config
├── documents
├── internal
│   ├── domain
│   │   ├── authentication
│   │   ├── shared
│   │   │   ├── errors
│   │   │   └── valueobjects
│   │   └── user
│   ├── infrastructure
│   │   ├── background
│   │   ├── db
│   │   │   ├── migrations
│   │   │   └── repository
│   │   ├── file_storage
│   │   ├── filestorage
│   │   ├── logging
│   │   ├── server
│   │   └── temporal
│   ├── interfaces
│   │   ├── persistence
│   │   └── rest
│   ├── middleware
│   ├── shared
│   └── usecases
├── pkg
│   └── utils

   ```

3. **Domain-Driven Design**
   - Model domain entities around business concepts
   - Use value objects for immutable concepts
   - Implement aggregate roots to maintain consistency boundaries
   - Define repository interfaces in the domain layer
   - Keep business logic in domain entities or domain services

## Code Quality Principles

4. **SOLID Principles**
   - **Single Responsibility**: Each struct/function should have only one reason to change
   - **Open/Closed**: Extend behavior through interfaces rather than modification
   - **Liskov Substitution**: Interface implementations must be substitutable
   - **Interface Segregation**: Prefer small, focused interfaces (1-3 methods)
   - **Dependency Inversion**: Depend on abstractions, not concrete implementations

5. **DRY (Don't Repeat Yourself)**
   - Extract common functionality into shared packages
   - Use middleware for cross-cutting concerns
   - Create utility functions for repeated operations
   - Implement generic repository patterns where appropriate

6. **Design Patterns**
   - **Repository Pattern**: Abstract data access behind interfaces
   - **Factory Pattern**: Create complex objects
   - **Strategy Pattern**: Switch algorithms at runtime
   - **Adapter Pattern**: Convert interfaces
   - **Decorator Pattern**: Add behavior dynamically
   - **Dependency Injection**: Provide dependencies from outside

## Development Approaches

7. **Test-Driven Development**
   - Write tests before implementation code
   - Start with test cases that validate interface contracts
   - Use table-driven tests for comprehensive test coverage
   - Use mocks for external dependencies
   - Run all tests after each change (`go test ./...`)

8. **Testing Practices**
   - Test each layer independently
   - Use interfaces to mock dependencies
   - Organize tests to mirror production code structure
   - Maintain test coverage above 80%
   - Write integration tests for critical paths

## Golang-Specific Best Practices

9. **Code Organization**
   - Group related functionality in packages
   - Use meaningful package names (avoid `util`, `common`, etc.)
   - Follow standard Go project layout
   - Keep packages small and focused

10. **Error Handling**
    - Use custom error types for domain-specific errors
    - Wrap errors with context using `fmt.Errorf("context: %w", err)`
    - Return errors rather than panic
    - Check all error returns
    - Use structured logging with errors

11. **Interfaces**
    - Define interfaces where behavior varies
    - Keep interfaces small (1-3 methods)
    - Define interfaces where they're used, not implemented
    - Use embedded interfaces to compose larger interfaces

12. **Concurrency**
    - Use goroutines and channels appropriately
    - Avoid shared memory; prefer communication
    - Use sync package for simple synchronization
    - Implement proper context propagation
    - Handle goroutine termination properly

13. **Performance**
    - Use pointers judiciously (only when mutation is needed)
    - Minimize allocations in hot paths
    - Use sync.Pool for frequently allocated objects
    - Benchmark performance-critical code
    - Profile before optimizing

## Implementation Guidelines

14. **Repository Layer**
    - Implement repository interfaces defined in domain
    - Handle database-specific errors and conversions
    - Use prepared statements for security
    - Implement proper transaction handling
    - Use context for timeouts and cancellation

15. **Use Case Layer**
    - Orchestrate domain operations
    - Validate input
    - Handle transaction boundaries
    - Implement business processes
    - Return domain objects (not persistence models)

16. **Handler Layer**
    - Convert between domain models and DTOs
    - Validate request parameters
    - Handle HTTP-specific concerns
    - Implement consistent error responses
    - Use middleware for cross-cutting concerns

17. **Configuration**
    - Use environment variables with sane defaults
    - Implement configuration validation
    - Support different environments (dev, test, prod)
    - Keep sensitive information in environment variables

## Specific Patterns

18. **Dependency Injection**
    ```go
    type Service struct {
        repo Repository
        logger Logger
    }
    

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FreePeak/cortex](https://github.com/FreePeak/cortex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
