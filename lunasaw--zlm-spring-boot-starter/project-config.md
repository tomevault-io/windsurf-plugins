---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **zlm-spring-boot-starter** project - a Spring Boot starter for integrating with ZLMediaKit streaming media
servers. It provides complete REST API encapsulation, Hook event handling, cluster management, and multiple load
balancing strategies for Java developers to easily integrate and manage ZLMediaKit streaming media servers.

## System Requirements

- Java 17+
- Spring Boot 3.5.3+
- ZLMediaKit server
- Jakarta EE specification (uses jakarta packages, not javax packages)

## Common Development Commands

### Build Commands

**Main Project:**

```bash
# Clean and build entire project
mvn clean install

# Run the main application (for development)
mvn spring-boot:run

# Package without running tests
mvn clean package -DskipTests

# Run with specific profile
mvn spring-boot:run -Dspring.profiles.active=dev
```

**Test Module (zlm-spring-boot-starter-test):**

```bash
# Run all tests
mvn test

# Run specific test class
mvn test -Dtest=NodeServiceTest

# Run tests with coverage
mvn test jacoco:report

# Integration tests
mvn test -Dtest=ZlmApiIntegrationTest
```

### Code Style Commands

```bash
# Format code using ali-code-style.xml
# Import ali-code-style.xml in Eclipse or use Eclipse Code Formatter plugin in IntelliJ

# Validate code style (if using checkstyle plugin)
mvn checkstyle:check
```

## Project Architecture

### Module Structure

The project follows a typical Spring Boot starter structure:

```
zlm-spring-boot-starter/
├── src/main/java/io/github/lunasaw/zlm/
│   ├── api/                    # REST API services and controllers
│   ├── config/                 # Auto-configuration and properties
│   ├── constant/              # API constants and enums
│   ├── entity/                # Data models and DTOs
│   ├── hook/                  # Hook event handling (controller, params, services)
│   ├── node/                  # Load balancing and node management
│   └── ZlmApplication.java    # Main application class
├── src/main/resources/
│   ├── META-INF/spring/       # Auto-configuration imports
│   └── application*.yml       # Configuration files
└── zlm-spring-boot-starter-test/  # Test module
```

### Core Components

**1. Auto-Configuration (`ZlmAutoConfiguration`)**

- Enables conditional bean creation based on `zlm.enable` property
- Auto-configures LoadBalancer, NodeSupplier, and ZlmHookService beans
- Supports 5 load balancing algorithms: random, round_robin, consistent_hashing, weight_round_robin, weight_random

**2. Node Management System**

- **NodeSupplier**: Interface for providing ZLM nodes (supports static config and dynamic discovery)
- **LoadBalancer**: Interface for load balancing algorithms
- **NodeService**: Unified service for node selection and management

**3. Hook Event System**

- **ZlmHookService**: Interface for handling ZLMediaKit callbacks
- **AbstractZlmHookService**: Base implementation with sensible defaults
- **ZlmHookController**: REST controller that receives hook callbacks

**4. REST API Integration**

- **ZlmRestService**: Static methods for calling ZLMediaKit REST APIs
- **ZlmApiController**: REST controller exposing ZLM APIs through HTTP endpoints

### Key Design Patterns

**Auto-Configuration Pattern**: Spring Boot starter pattern with conditional bean creation
**Strategy Pattern**: Multiple load balancing implementations
**Template Method Pattern**: AbstractZlmHookService provides default implementations
**Supplier Pattern**: NodeSupplier for flexible node discovery
**Service Layer Pattern**: NodeService abstracts load balancing complexity

## Configuration

### Basic Configuration

```yaml
zlm:
  enable: true                    # Enable/disable ZLM functionality
  balance: round_robin           # Load balancing algorithm
  nodes:                         # Static node configuration
    - server-id: zlm-node-1
      host: "http://127.0.0.1:9092"
      secret: zlm
      enabled: true
      hook-enabled: true
      weight: 1                  # For weighted algorithms
```

### Dynamic Node Discovery

Implement `NodeSupplier` interface for dynamic node discovery from databases, registries, or config centers:

```java
@Component
public class DatabaseNodeSupplier implements NodeSupplier {
    @Override
    public List<ZlmNode> getNodes() {
        // Return dynamic node list
    }
}
```

## Testing Strategy

### Test Module Structure

Tests are located in the separate `zlm-spring-boot-starter-test` module to avoid circular dependencies.

**Test Categories:**

- **Unit Tests**: `ZlmServiceUnitTest`, `ZlmHookParamTest`
- **Integration Tests**: `ZlmApiIntegrationTest`, `ZlmAutoConfigurationTest`
- **Service Tests**: `NodeServiceTest`, `ZlmApiControllerTest`

**Test Configuration:**

- Uses Spring Boot Test with `@SpringBootTest(classes = ZlmAutoConfiguration.class)`
- JUnit 4 with SpringRunner
- Mockito for mocking dependencies

### Running Tests

```bash
# Run all tests
mvn test

# Run specific test category
mvn test -Dtest="*UnitTest"
mvn test -Dtest="*IntegrationTest"

# Run tests with specific profile
mvn test -Dspring.profiles.active=test
```

## Key APIs and Usage Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lunasaw/zlm-spring-boot-starter](https://github.com/lunasaw/zlm-spring-boot-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
