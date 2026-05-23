---
trigger: always_on
description: Best practices for local testing in Spring Boot applications using `spring-boot-docker-compose` for seamless integration with external services like databases, message queues, and caches.
---

# Spring Boot Local Testing with Docker Compose

Best practices for local testing in Spring Boot applications using `spring-boot-docker-compose` for seamless integration with external services like databases, message queues, and caches.

## Implementing These Principles

These guidelines are built upon the following core principles:

- **Seamless Integration**: Use spring-boot-docker-compose to automatically manage external service dependencies
- **Environment Parity**: Maintain consistency between local development and production environments
- **Test Isolation**: Ensure tests are independent and can run in any order without side effects
- **Performance Optimization**: Minimize container startup time and resource usage for faster development cycles
- **Configuration Management**: Use profiles and dynamic properties for flexible environment-specific configurations

## Table of contents

- Rule 1: Dependency Configuration
- Rule 2: Docker Compose Service Definition
- Rule 3: Application Profile Configuration
- Rule 4: Integration Test Setup
- Rule 5: Service Connection Management
- Rule 6: Health Check Implementation
- Rule 7: Test Data Management
- Rule 8: Performance Optimization

## Rule 1: Dependency Configuration

Title: Proper Spring Boot Docker Compose Dependency Setup
Description: Configure the spring-boot-docker-compose dependency correctly for runtime-only usage to automatically manage Docker services during application startup.

**Good example:**

```xml
<!-- Maven -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-docker-compose</artifactId>
    <scope>runtime</scope>
</dependency>

<!-- Testcontainers for integration tests -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-testcontainers</artifactId>
    <scope>test</scope>
</dependency>
```

**Bad Example:**

```xml
<!-- Don't include as compile dependency -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-docker-compose</artifactId>
    <scope>compile</scope>
</dependency>

<!-- Missing testcontainers dependency -->
```

## Rule 2: Docker Compose Service Definition

Title: Well-structured Docker Compose Configuration
Description: Define services with proper health checks, environment variables, and port mappings for reliable local testing.

**Good example:**

```yaml
# compose.yaml
services:
  postgres:
    image: 'postgres:15'
    environment:
      POSTGRES_DB: testdb
      POSTGRES_USER: testuser
      POSTGRES_PASSWORD: testpass
    ports:
      - '5432:5432'
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U testuser -d testdb"]
      interval: 10s
      timeout: 5s
      retries: 5
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

**Bad Example:**

```yaml
# compose.yaml
services:
  postgres:
    image: 'postgres'  # No version specified
    environment:
      - POSTGRES_PASSWORD=password  # Hardcoded, no DB/USER
    # Missing health checks
    # Missing volume persistence
    # Missing proper port mapping
```

## Rule 3: Application Profile Configuration

Title: Environment-specific Configuration Management
Description: Use Spring profiles to manage different configurations for local development, testing, and production environments.

**Good example:**

```yaml
# application-local.yml
spring:
  docker:
    compose:
      enabled: true
      file: compose.yaml
      lifecycle-management: start_and_stop
      readiness:
        wait: HEALTHY
        timeout: 2m
  
  datasource:
    url: jdbc:postgresql://localhost:5432/testdb
    username: testuser
    password: testpass
    
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true

logging:
  level:
    org.springframework.boot.docker: DEBUG
```

**Bad Example:**

```yaml
# application.yml
spring:
  docker:
    compose:
      enabled: true  # Always enabled, no profile separation
      lifecycle-management: none  # No lifecycle management
  
  datasource:
    url: jdbc:postgresql://localhost:5432/proddb  # Production DB in local config
    username: root  # Unsafe credentials
    password: admin
```

## Rule 4: Integration Test Setup

Title: Testcontainers Integration with Service Connections
Description: Use @ServiceConnection and @Testcontainers for clean integration test setup with automatic container management.

**Good example:**

```java
@SpringBootTest
@Testcontainers
@ActiveProfiles("test")
class UserRepositoryIntegrationTest {
    
    @Container
    @ServiceConnection
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void shouldSaveAndRetrieveUser() {
        User user = new User("john.doe@example.com", "John Doe");
        User saved = userRepository.save(user);
        
        assertThat(saved.getId()).isNotNull();
        assertThat(userRepository.findByEmail("john.doe@example.com"))
                .isPresent()
                .get()

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jabrena/cursor-rules-spring-boot](https://github.com/jabrena/cursor-rules-spring-boot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
