---
trigger: always_on
description: This file provides context for agentic coding tools operating in this repository.
---

# AGENTS.md — Coding Agent Guidelines

This file provides context for agentic coding tools operating in this repository.

---

## Project Overview

A multi-module Maven project demonstrating a Bitcoin price streaming system using Spring Boot, Apache Kafka (via Spring Cloud Stream), WebSocket, Thymeleaf, Spring Security, Swagger/OpenAPI (springdoc), and MySQL.

- **Java version:** 25
- **Spring Boot:** 4.0.6
- **Spring Cloud:** 2025.1.1
- **Modules:** `bitcoin-api` (REST API + Kafka producer + JPA/MySQL), `bitcoin-client` (Kafka consumer + WebSocket + Thymeleaf UI + Security)
- **Root package:** `com.ivanfranchin`

---

## Build & Test Commands

All commands use the Maven wrapper (`./mvnw`). There is no Makefile, Gradle, or custom lint runner.

### Build

```bash
# Build all modules
./mvnw clean package

# Build skipping tests
./mvnw clean package -DskipTests
```

### Run Tests

```bash
# Run all tests (all modules)
./mvnw test

# Run tests for a single module
./mvnw test --projects bitcoin-api
./mvnw test --projects bitcoin-client

# Run a single test class
./mvnw test --projects bitcoin-api -Dtest=PriceControllerTests

# Run a single test method
./mvnw test --projects bitcoin-api -Dtest=PriceControllerTests#testGetLastPriceReturnsLatestBitcoinPrice
```

### Run Applications Locally

Both apps require a running MySQL + Kafka (start with `docker compose up -d`).

```bash
./mvnw clean spring-boot:run --projects bitcoin-api -Dspring-boot.run.jvmArguments="-Dserver.port=9081"
./mvnw clean spring-boot:run --projects bitcoin-client -Dspring-boot.run.jvmArguments="-Dserver.port=9082"
```

### Docker

```bash
# Start infrastructure (MySQL, Kafka, Kafdrop)
docker compose up -d
docker compose down -v

# Build Docker images for both modules
./build-docker-images.sh

# Run/stop apps as Docker containers
./start-apps.sh
./stop-apps.sh

# Remove Docker images for both modules
./remove-docker-images.sh
```

---

## Project Structure

```
springboot-kafka-websocket/         ← Root multi-module Maven project
├── pom.xml                         ← Parent POM
├── compose.yaml
├── bitcoin-api/                    ← Module 1
│   └── src/main/java/com/ivanfranchin/bitcoinapi/
│       ├── config/                 ← @Configuration classes
│       ├── price/                  ← Feature package (domain slice)
│       │   ├── dto/                ← Records used as response types
│       │   ├── event/              ← Records used as Kafka event types
│       │   └── model/              ← JPA @Entity classes
│       └── runner/                 ← ApplicationRunner implementations
└── bitcoin-client/                 ← Module 2
    └── src/main/java/com/ivanfranchin/bitcoinclient/
        ├── controller/             ← @Controller (HTTP + @MessageMapping chat handler)
        ├── kafka/                  ← Kafka consumer + event record
        ├── security/
        └── websocket/              ← WebSocketConfig + ChatMessage record
```

Package structure follows **feature/domain slicing** (e.g., `price/`, `kafka/`, `websocket/`), with sub-packages `dto/`, `event/`, `model/` inside feature packages where needed.

---

## Code Style Guidelines

### Indentation & Formatting

- **2 spaces (Google Java Format default)** — never tabs
- Opening brace on the **same line** as the declaration (K&R style)
- One blank line between methods; one blank line after the class opening brace before the first member
- No enforced line-length limit, but keep lines readable

### Import Organization

Imports follow Google Java Format + custom ordering: static imports first (separated by a blank line), then non-static imports sorted lexicographically with order `java.*` → `jakarta.*` → `org.*` → `com.*`. Wildcard imports are forbidden. Static imports are allowed in `src/main` with Spotless enforcement.

```java
import static org.assertj.core.api.Assertions.assertThat;

import java.math.BigDecimal;
import java.time.LocalDateTime;

import jakarta.persistence.Entity;

import org.springframework.stereotype.Component;

import com.ivanfranchin.bitcoinapi.price.model.Price;
```

| Element | Convention | Example |
|---|---|---|
| Classes / Records | `PascalCase` | `PriceController`, `PriceChanged` |
| Methods | `camelCase` | `getLastPrice()`, `streamNewPrice()` |
| Local variables | `camelCase` | `priceMessage`, `currentPrice` |
| Constants (`static final`) | `UPPER_SNAKE_CASE` | `BINDING_NAME` |
| Packages | lowercase, domain-segment style | `com.ivanfranchin.bitcoinapi.price.dto` |
| Test classes | `[TestedClass]Tests` | `PriceControllerTests` |

### Annotations

- One annotation **per line**, stacked above the declaration
- **Class annotation ordering:** Lombok annotations first, then Spring stereotype/config annotations last:

```java
@Slf4j
@RequiredArgsConstructor
@Component
public class PriceEventEmitter { ... }

@RequiredArgsConstructor
@RestController
@RequestMapping("/api/bitcoin")
public class PriceController { ... }

@EnableScheduling
@Configuration
public class SchedulingConfig { ... }
```

- `@Bean` methods are **package-private** (no `public` modifier):

```java
@Bean
OpenAPI customOpenAPI() { ... }

@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http) { ... }
```

### Dependency Injection


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivangfr/springboot-kafka-websocket](https://github.com/ivangfr/springboot-kafka-websocket) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
