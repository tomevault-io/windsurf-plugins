---
trigger: always_on
description: This document outlines the security best practices for the OpenFrame project.
---

# Security Practices

This document outlines the security best practices for the OpenFrame project.

## Authentication and Authorization

### JWT Authentication

OpenFrame uses JWT (JSON Web Tokens) for authentication:

```java
@Configuration
@EnableWebFluxSecurity
public class SecurityConfig {
    @Bean
    public SecurityWebFilterChain securityWebFilterChain(ServerHttpSecurity http) {
        return http
            .csrf().disable()
            .authorizeExchange()
                .pathMatchers("/auth/**").permitAll()
                .pathMatchers("/actuator/health").permitAll()
                .pathMatchers("/actuator/**").hasRole("ADMIN")
                .anyExchange().authenticated()
            .and()
            .addFilterAt(jwtAuthenticationFilter, SecurityWebFiltersOrder.AUTHENTICATION)
            .build();
    }
    
    @Bean
    public ReactiveJwtDecoder jwtDecoder(@Value("${openframe.security.jwt.public-key}") RSAPublicKey publicKey) {
        return NimbusReactiveJwtDecoder.withPublicKey(publicKey).build();
    }
}
```

### Role-Based Access Control

Implement role-based access control (RBAC):

```java
@RestController
@RequestMapping("/api/admin")
public class AdminController {
    private final AdminService adminService;
    
    @GetMapping("/users")
    @PreAuthorize("hasRole('ADMIN')")
    public Flux<User> getAllUsers() {
        return adminService.getAllUsers();
    }
    
    @PostMapping("/users/{id}/roles")
    @PreAuthorize("hasRole('ADMIN')")
    public Mono<User> updateUserRoles(@PathVariable String id, @RequestBody List<String> roles) {
        return adminService.updateUserRoles(id, roles);
    }
}
```

### Permission Checks

Implement fine-grained permission checks:

```java
@Service
public class DeviceService {
    private final DeviceRepository deviceRepository;
    private final SecurityService securityService;
    
    public Mono<Device> getDeviceById(String id) {
        return deviceRepository.findById(id)
            .filterWhen(device -> securityService.hasPermission("READ", "DEVICE", device.getId()));
    }
    
    public Mono<Device> updateDevice(String id, Device device) {
        return deviceRepository.findById(id)
            .filterWhen(existingDevice -> securityService.hasPermission("WRITE", "DEVICE", id))
            .flatMap(existingDevice -> {
                // Update device
                return deviceRepository.save(device);
            });
    }
}
```

## Input Validation

### Request Validation

Validate all incoming requests:

```java
@RestController
@RequestMapping("/api/devices")
public class DeviceController {
    private final DeviceService deviceService;
    private final Validator validator;
    
    @PostMapping
    public Mono<ResponseEntity<Device>> createDevice(@RequestBody @Valid DeviceRequest request) {
        return deviceService.createDevice(request.toDevice())
            .map(device -> ResponseEntity.status(HttpStatus.CREATED).body(device));
    }
    
    @Data
    public static class DeviceRequest {
        @NotBlank(message = "Hostname is required")
        @Size(min = 3, max = 50, message = "Hostname must be between 3 and 50 characters")
        private String hostname;
        
        @NotBlank(message = "Operating system is required")
        private String operatingSystem;
        
        @Pattern(regexp = "^([0-9]{1,3}\\.){3}[0-9]{1,3}$", message = "Invalid IP address format")
        private String ipAddress;
        
        public Device toDevice() {
            Device device = new Device();
            device.setHostname(hostname);
            device.setOperatingSystem(operatingSystem);
            device.setIpAddress(ipAddress);
            return device;
        }
    }
}
```

### Input Sanitization

Sanitize user input to prevent XSS:

```java
@Component
public class InputSanitizer {
    private final PolicyFactory policy = new HtmlPolicyBuilder()
        .allowElements("b", "i", "u", "strong", "em")
        .allowUrlProtocols("https")
        .allowAttributes("href").onElements("a")
        .requireRelNofollowOnLinks()
        .toFactory();
    
    public String sanitize(String input) {
        if (input == null) {
            return null;
        }
        return policy.sanitize(input);
    }
}

@Service
public class NoteService {
    private final NoteRepository noteRepository;
    private final InputSanitizer inputSanitizer;
    
    public Mono<Note> createNote(Note note) {
        note.setTitle(inputSanitizer.sanitize(note.getTitle()));
        note.setContent(inputSanitizer.sanitize(note.getContent()));
        return noteRepository.save(note);
    }
}
```

## Secure Communication

### TLS Configuration

Configure TLS for secure communication:

```java
@Configuration
public class TlsConfig {
    @Bean
    public NettyServerCustomizer nettyServerCustomizer(
            @Value("${server.ssl.key-store}") String keyStore,
            @Value("${server.ssl.key-store-password}") String keyStorePassword) {
        return httpServer -> {
            SslContext sslContext = SslContextBuilder.forServer(
                    new File(keyStore),
                    keyStorePassword)
                .protocols("TLSv1.2", "TLSv1.3")
                .ciphers(List.of(

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flamingo-stack/openframe-oss-tenant](https://github.com/flamingo-stack/openframe-oss-tenant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
