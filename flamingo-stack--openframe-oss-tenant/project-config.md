---
trigger: always_on
description: description: API development patterns and best practices for OpenFrame REST and GraphQL APIs
---


---
description: API development patterns and best practices for OpenFrame REST and GraphQL APIs
globs:
  - "openframe/services/*/src/main/java/**/controller/**"
  - "openframe/services/*/src/main/java/**/service/**"
  - "openframe/libs/api-library/**"
alwaysApply: false
---

# API Development Patterns in OpenFrame

OpenFrame follows a shared library approach to eliminate code duplication between GraphQL and REST APIs. Follow these patterns for consistent API development.

## Shared Library Architecture

### Business Logic in api-library
- **DO**: Place all business logic in [api-library services](mdc:openframe/libs/api-library/src/main/java/com/openframe/api/service)
- **DON'T**: Duplicate business logic between GraphQL and REST controllers
- **Pattern**: Controllers are thin adapters that delegate to shared services

```java
// ✅ GOOD - Service in api-library
@Service
public class DeviceService {
    public DeviceQueryResult queryDevices(DeviceFilterCriteria criteria) {
        // Business logic here
    }
}

// ✅ GOOD - REST controller delegates to service
@RestController
public class DeviceController {
    private final DeviceService deviceService;

    @GetMapping("/devices")
    public DeviceResponse getDevices(@AuthenticationPrincipal AuthPrincipal principal) {
        return deviceService.queryDevices(criteria);
    }
}
```

## Controller Patterns

### Modern Spring Boot Style
Use DTO + Exceptions approach instead of ResponseEntity everywhere:

```java
// ✅ GOOD - Modern Spring Boot style
@GetMapping("/{id}")
@ResponseStatus(OK)
public DeviceResponse getDevice(@PathVariable String id) {
    Device device = deviceService.findById(id)
        .orElseThrow(() -> new DeviceNotFoundException("Device not found: " + id));
    return deviceMapper.toResponse(device);
}

// ❌ BAD - Old ResponseEntity style
@GetMapping("/{id}")
public ResponseEntity<DeviceResponse> getDevice(@PathVariable String id) {
    return deviceService.findById(id)
        .map(device -> ResponseEntity.ok(deviceMapper.toResponse(device)))
        .orElse(ResponseEntity.notFound().build());
}
```

### Authentication Principal Pattern
Always use `@AuthenticationPrincipal AuthPrincipal principal` for user context:

```java
@RestController
@RequestMapping("/api-keys")
public class ApiKeyController {
    @GetMapping
    public List<ApiKeyResponse> getApiKeys(@AuthenticationPrincipal AuthPrincipal principal) {
        return apiKeyService.getApiKeysForUser(principal.getId());
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public CreateApiKeyResponse createApiKey(
            @Valid @RequestBody CreateApiKeyRequest request,
            @AuthenticationPrincipal AuthPrincipal principal) {
        return apiKeyService.createApiKey(principal.getId(), request);
    }
}
```

**Reference**: [ApiKeyController.java](mdc:openframe/services/openframe-api/src/main/java/com/openframe/api/controller/ApiKeyController.java)

## Error Handling Patterns

### Global Exception Handler
Use global exception handlers with consistent error responses:

```java
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(DeviceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleDeviceNotFound(DeviceNotFoundException ex) {
        return ErrorResponse.builder()
            .code("DEVICE_NOT_FOUND")
            .message(ex.getMessage())
            .timestamp(LocalDateTime.now())
            .build();
    }
}
```

### Custom Domain Exceptions
Create specific exceptions for domain errors:

```java
public class ApiKeyNotFoundException extends RuntimeException {
    public ApiKeyNotFoundException(String keyId) {
        super("API key not found: " + keyId);
    }
}
```

**Reference**: [ErrorResponse.java](mdc:openframe/libs/openframe-core/src/main/java/com/openframe/core/dto/ErrorResponse.java)

## DTO and Mapping Patterns

### Record-based DTOs
Use Java records for immutable DTOs:

```java
public record CreateApiKeyRequest(
    @NotBlank String name,
    String description,
    @Future LocalDateTime expiresAt
) {}

public record ApiKeyResponse(
    String id,
    String name,
    String description,
    boolean enabled,
    LocalDateTime createdAt,
    LocalDateTime expiresAt
) {}
```

### Service to DTO Conversion
Convert between domain models and DTOs in dedicated mappers:

```java
@Component
public class DeviceMapper {
    public DeviceResponse toDeviceResponse(DeviceQueryResult result) {
        return DeviceResponse.builder()
            .id(result.getId())
            .hostname(result.getHostname())
            .status(result.getStatus())
            .build();
    }
}
```

## Validation Patterns

### Bean Validation
Use Bean Validation annotations for request validation:

```java
public record DeviceRequest(
    @NotBlank(message = "Hostname is required")
    @Size(min = 3, max = 50, message = "Hostname must be between 3 and 50 characters")
    String hostname,

    @NotBlank(message = "Operating system is required")
    String operatingSystem,

    @Pattern(regexp = "^([0-9]{1,3}\\.){3}[0-9]{1,3}$", message = "Invalid IP address format")
    String ipAddress
) {}
```

### Controller Validation
Use `@Valid` annotation in controller methods:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flamingo-stack/openframe-oss-tenant](https://github.com/flamingo-stack/openframe-oss-tenant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
