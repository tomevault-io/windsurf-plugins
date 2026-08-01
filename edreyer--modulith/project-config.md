---
trigger: always_on
description: This project is a Kotlin-based Spring Boot application that follows a hexagonal architecture combined with Domain-Driven Design (DDD) principles. It's designed as a "Modulith" - essentially a microservice architecture deployed as a monolith, providing the benefits of microservices without the operational complexity.
---

# Project Coding Guidelines

## Overview
This project is a Kotlin-based Spring Boot application that follows a hexagonal architecture combined with Domain-Driven Design (DDD) principles. It's designed as a "Modulith" - essentially a microservice architecture deployed as a monolith, providing the benefits of microservices without the operational complexity.

The codebase emphasizes functional programming, type safety, and clean separation of concerns. It uses Kotlin's advanced features like coroutines, context receivers, and algebraic data types to create a robust and maintainable codebase.

## Code Style

### Comments and Documentation
- Use KDoc comments for public APIs and important internal components
- Use single-line comments for implementation details
- Document non-obvious behavior and design decisions
- Include examples in documentation where helpful

```kotlin
/**
 * Delegate UserData class to cut down on copy-pasta in each ADT instance of User
 * see: https://proandroiddev.com/simpler-kotlin-class-hierarchies-using-class-delegation-35464106fed5
 */
internal interface UserFields {
  val id: UserId
  val msisdn: Msisdn
  val email: EmailAddress
  val encryptedPassword: NonEmptyString
}
```

## Naming Conventions

### General Naming Rules
- Use descriptive, meaningful names
- Avoid abbreviations except for common ones (e.g., ID, DTO)
- Use domain terminology consistently

### Class and Interface Naming
- Use PascalCase for class and interface names
- Use nouns for entity classes (User, Appointment)
- Use adjective+noun for state-based classes (ActiveUser, UnregisteredUser)
- Use verb+noun+suffix for workflow classes (RegisterUserWorkflow, CancelAppointmentWorkflow)
- Use noun+suffix for interfaces (FindUserPort, UserEventPort)

```kotlin
// Class naming examples
sealed class User
data class ActiveUser(private val data: UserData) : User(), UserFields by data
internal class RegisterUserWorkflow(/*...*/) : BaseSafeWorkflow<RegisterUserCommand, UserRegisteredEvent>()
internal interface FindUserPort
```

### Function and Method Naming
- Use camelCase for function and method names
- Use verbs for functions that perform actions
- Use "get" prefix for accessor methods
- Use "is/has/can" prefix for boolean methods
- Use "to" prefix for conversion methods

```kotlin
// Function naming examples
fun findUserById(userId: String): User?
fun isValid(value: String): Boolean
fun toUserDto(): UserDto
```

### Variable Naming
- Use camelCase for variable names
- Use descriptive names that indicate purpose
- Use single-letter names only for local variables with limited scope (e.g., loop counters)

```kotlin
// Variable naming examples
val userRepository: UserRepository
val passwordEncoder: PasswordEncoder
for (i in items.indices) { /*...*/ }
```

### File Naming
- Use PascalCase for file names containing a single primary class
- Match the file name to the primary class name
- Use plural for files containing multiple related classes

```kotlin
// File naming examples
User.kt         // Contains User class and related classes
Workflows.kt    // Contains multiple workflow classes
SimpleTypes.kt  // Contains multiple simple type classes
```

### Package Naming
- Use lowercase with dots as separators
- Follow the hexagonal architecture structure:
  - domain: Core domain models
  - application: Application services, ports, and workflows
  - adapter.in: Input adapters (web controllers, etc.)
  - adapter.out: Output adapters (persistence, external services, etc.)

```kotlin
// Package naming examples
io.liquidsoftware.base.user.domain
io.liquidsoftware.base.user.application.workflows
io.liquidsoftware.base.user.adapter.in.web
io.liquidsoftware.base.user.adapter.out.persistence
```

## Architecture Patterns

### Hexagonal Architecture
The project follows a hexagonal architecture (also known as ports and adapters) with the following components:

- **Domain Layer**: Contains the core domain models and business logic
  - Located in the `domain` package
  - Has no dependencies on other layers
  - Uses algebraic data types (sealed classes) to model domain entities

- **Application Layer**: Contains the application services, ports, and workflows
  - Located in the `application` package
  - Depends only on the domain layer
  - Defines ports (interfaces) for communication with adapters
  - Implements workflows that orchestrate domain logic

- **Adapter Layer**: Contains the adapters that connect to external systems
  - Located in the `adapter` package
  - Divided into `in` (driving) and `out` (driven) adapters
  - Implements the ports defined by the application layer

```kotlin
// Domain model example
internal sealed class User: UserFields {
  // Domain logic
}

// Application port example
internal interface FindUserPort {
  suspend fun findUserById(userId: String): User?
}

// Adapter implementation example
internal class UserPersistenceAdapter(
  private val userRepository: UserRepository,
  private val ac: AclChecker
) : FindUserPort, UserEventPort {
  // Implementation
}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [edreyer/modulith](https://github.com/edreyer/modulith) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
