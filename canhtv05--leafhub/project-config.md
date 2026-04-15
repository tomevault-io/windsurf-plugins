---
trigger: always_on
description: This is a microservices-based Spring Boot application with the following modules:
---

# Cursor Rules for MS Project

## Project Overview
This is a microservices-based Spring Boot application with the following modules:
- **auth**: Authentication and authorization service (JPA/PostgreSQL)
- **profile**: User profile service (Neo4j)
- **file**: File upload and management service (MongoDB)
- **notification**: Email notification service
- **graphql-bff**: GraphQL BFF layer using Netflix DGS
- **common**: Shared common code and gRPC definitions
- **gateway**: API Gateway

## Code Style & Conventions

### Naming Conventions

#### Domain Entity Naming
- Use PascalCase: `User`, `UserProfile`, `Interest`, `File`

#### DTO Naming
- **General DTO**: `{Domain}DTO` (e.g., `UserDTO`, `InterestDTO`, `UserProfileDTO`)
- **Request DTO**: `Update{Domain}{Field}Req` or `{Action}{Domain}{Field}Req` (e.g., `UpdateBioProfileReq`, `ChangeCoverByUrlReq`, `UserProfileCreationReq`)
- **Response DTO**: `{Domain}Res` or `Update{Domain}{Field}Res` (e.g., `UserProfileRes`, `UserRes`, `FileRes`)

#### Service Naming
- **Service**: `{Domain}Service` (e.g., `UserService`, `UserProfileService`, `InterestService`, `FileService`)

#### Repository Naming
- **Repository**: `{Domain}Repository` (e.g., `UserRepository`, `UserProfileRepository`, `InterestRepository`, `FileRepository`)

### Package Structure
- Follow the pattern: `com.leaf.{module}.{layer}`
- Layers: `controller`, `service`, `repository`, `domain`, `dto`, `grpc`, `config`, `security`, `exception`
- DTOs should be in subpackages: `dto.req`, `dto.res`, `dto.search`, `dto.excel` when applicable

### Service Layer Patterns

#### Service Class Structure
```java
@Slf4j
@Service
@RequiredArgsConstructor
@Transactional(propagation = Propagation.REQUIRED, rollbackFor = Exception.class)
public class UserService {
    // Dependencies injected via constructor (Lombok @RequiredArgsConstructor)
    private final UserRepository repository;
    private final OtherService otherService;

    // Write operations
    public UserDTO createUser(UserProfileCreationReq req) {
        // Implementation
    }

    // Read operations
    @Transactional(readOnly = true)
    public UserDTO getUser(Long id) {
        // Implementation
    }
}
```

**Naming**: Service class name follows pattern `{Domain}Service` (e.g., `UserService`, `UserProfileService`, `InterestService`)

#### Transaction Management
- **Class-level**: Use `@Transactional(propagation = Propagation.REQUIRED, rollbackFor = Exception.class)` for services with write operations
  - Import: `import org.springframework.transaction.annotation.Propagation;`
  - Write operations will inherit transaction settings from class-level annotation
- **Write operations**: No need to add `@Transactional` annotation to write methods (inherit from class-level)
- **Read operations**: Always add `@Transactional(readOnly = true)` to each read method to override class-level settings
- **Propagation**: Use `REQUIRED` (default) for most cases

#### Exception Handling
- Always use `ApiException` with `ErrorMessage` enum
- Pattern: `throw new ApiException(ErrorMessage.XXX)`
- Use `SecurityUtils.getCurrentUserLogin().orElseThrow(() -> new ApiException(ErrorMessage.UNAUTHENTICATED))`
- Use `repository.findById(id).orElseThrow(() -> new ApiException(ErrorMessage.XXX_NOT_FOUND))`

### Controller Layer Patterns

#### REST Controller Structure
```java
@RestController
@RequiredArgsConstructor
public class UserController {
    private final UserService service;

    @GetMapping("/users")
    public ResponseEntity<ResponseObject<UserDTO>> getUser() {
        return ResponseEntity.ok(ResponseObject.success(service.getUser()));
    }

    @PostMapping("/users")
    public ResponseEntity<ResponseObject<UserDTO>> createUser(@RequestBody UserProfileCreationReq req) {
        return ResponseEntity.ok(ResponseObject.success(service.createUser(req)));
    }
}
```

#### Response Format
- Always return `ResponseEntity<ResponseObject<T>>`
- Use `ResponseObject.success(data)` for success responses
- Use `ResponseObject.success(data, pageResponse)` for paginated responses
- Use `ResponseObject.error(ErrorMessage.XXX)` for error responses

### DTO Patterns

#### DTO Requirements
- **All DTOs must implement `Serializable`**
- **All DTOs must have `private static final long serialVersionUID = 1L;`**
- **General DTOs (e.g., `UserDTO`, `UserProfileDTO`) must have static factory method `toDTO(Entity entity)` using `BeanUtils.copyProperties`**
- **Response DTOs must have static factory method using `BeanUtils.copyProperties`**

#### General DTO Structure
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class UserDTO implements Serializable {
    private static final long serialVersionUID = 1L;

    private Long id;
    private String username;
    private String email;

    // Static factory method for entity conversion using BeanUtils
    public static UserDTO toDTO(User entity) {
        UserDTO dto = new UserDTO();
        BeanUtils.copyProperties(entity, dto);
        return dto;
    }
}
```

**Note**: Don't forget to import: `import org.springframework.beans.BeanUtils;`

**Naming**: General DTO follows pattern `{Domain}DTO` (e.g., `UserDTO`, `InterestDTO`, `UserProfileDTO`)

#### Response DTO Structure
```java
@Data

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/canhtv05) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
