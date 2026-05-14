---
trigger: always_on
description: Cursor rules for Kotlin development with Ktor integration.
---

## Instruction to developer: save this file as .cursorrules and place it on the root project directory

## Core Principles
- Follow **SOLID**, **DRY**, **KISS**, and **YAGNI** principles
- Adhere to **OWASP** security best practices
- Break tasks into smallest units and solve problems step-by-step

## Technology Stack
- **Framework**: Kotlin Ktor with Kotlin 2.1.20+
- **JDK**: 21 (LTS)
- **Build**: Gradle with Kotlin DSL
- **Dependencies**: Ktor Server Core/Netty, kotlinx.serialization, Exposed, HikariCP, kotlin-logging, Koin, Kotest

## Application Structure (Feature-Based)
- **Organize by business features, not technical layers**
- Each feature is self-contained with all related components
- Promotes modularity, reusability, and better team collaboration
- Makes codebase easier to navigate and maintain
- Enables parallel development on different features
```
src/main/kotlin/com/company/app/
├── common/              # Shared utilities, extensions
├── config/              # Application configuration, DI
└── features/
    ├── auth/            # Feature directory
    │   ├── models/
    │   ├── repositories/
    │   ├── services/
    │   └── routes/
    └── users/           # Another feature
        ├── ...
```

Test structure mirrors the feature-based organization:
```
src/test/kotlin/com/company/app/
├── common/
└── features/
    ├── auth/
    │   ├── models/
    │   ├── repositories/
    │   ├── services/
    │   └── routes/
    └── users/
        ├── ...
```

## Application Logic Design
1. Route handlers: Handle requests/responses only
2. Services: Contain business logic, call repositories
3. Repositories: Handle database operations
4. Entity classes: Data classes for database models
5. DTOs: Data transfer between layers

## Entities & Data Classes
- Use Kotlin data classes with proper validation
- Define Table objects when using Exposed ORM
- Use UUID or auto-incrementing integers for IDs

## Repository Pattern
```kotlin
interface UserRepository {
    suspend fun findById(id: UUID): UserDTO?
    suspend fun create(user: CreateUserRequest): UserDTO
    suspend fun update(id: UUID, user: UpdateUserRequest): UserDTO?
    suspend fun delete(id: UUID): Boolean
}

class UserRepositoryImpl : UserRepository {
    override suspend fun findById(id: UUID): UserDTO? = withContext(Dispatchers.IO) {
        transaction {
            Users.select { Users.id eq id }
                .mapNotNull { it.toUserDTO() }
                .singleOrNull()
        }
    }
    // Other implementations...
}
```

## Service Layer
```kotlin
interface UserService {
    suspend fun getUserById(id: UUID): UserDTO
    suspend fun createUser(request: CreateUserRequest): UserDTO
    suspend fun updateUser(id: UUID, request: UpdateUserRequest): UserDTO
    suspend fun deleteUser(id: UUID)
}

class UserServiceImpl(
    private val userRepository: UserRepository
) : UserService {
    override suspend fun getUserById(id: UUID): UserDTO {
        return userRepository.findById(id) ?: throw ResourceNotFoundException("User", id.toString())
    }
    // Other implementations...
}
```

## Route Handlers
```kotlin
fun Application.configureUserRoutes(userService: UserService) {
    routing {
        route("/api/users") {
            get("/{id}") {
                val id = call.parameters["id"]?.let { UUID.fromString(it) }
                    ?: throw ValidationException("Invalid ID format")
                val user = userService.getUserById(id)
                call.respond(ApiResponse("SUCCESS", "User retrieved", user))
            }
            // Other routes...
        }
    }
}
```

## Error Handling
```kotlin
open class ApplicationException(
    message: String,
    val statusCode: HttpStatusCode = HttpStatusCode.InternalServerError
) : RuntimeException(message)

class ResourceNotFoundException(resource: String, id: String) :
    ApplicationException("$resource with ID $id not found", HttpStatusCode.NotFound)

fun Application.configureExceptions() {
    install(StatusPages) {
        exception<ResourceNotFoundException> { call, cause ->
            call.respond(cause.statusCode, ApiResponse("ERROR", cause.message ?: "Resource not found"))
        }
        exception<Throwable> { call, cause ->
            call.respond(HttpStatusCode.InternalServerError, ApiResponse("ERROR", "An internal error occurred"))
        }
    }
}
```

## Testing Strategies and Coverage Requirements

### Test Coverage Requirements
- **Minimum coverage**: 80% overall code coverage required
- **Critical components**: 90%+ coverage for repositories, services, and validation
- **Test all edge cases**: Empty collections, null values, boundary conditions
- **Test failure paths**: Exception handling, validation errors, timeouts
- **All public APIs**: Must have integration tests
- **Performance-critical paths**: Must have benchmarking tests

### Unit Testing with Kotest
```kotlin
class UserServiceTest : DescribeSpec({
    describe("UserService") {
        val mockRepository = mockk<UserRepository>()
        val userService = UserServiceImpl(mockRepository)

        it("should return user when exists") {
            val userId = UUID.randomUUID()
            val user = UserDTO(userId.toString(), "Test User", "test@example.com")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XD3an/awesome-ai-coding-all-in-one](https://github.com/XD3an/awesome-ai-coding-all-in-one) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
