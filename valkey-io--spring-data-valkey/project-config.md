---
trigger: always_on
description: This file provides AI agents with the minimum but sufficient context to work productively in the Spring Data Valkey repository. It covers build commands, contribution requirements, and essential guardrails for maintaining code quality.
---

# AGENTS: Unified Context for Agentic Tools

This file provides AI agents with the minimum but sufficient context to work productively in the Spring Data Valkey repository. It covers build commands, contribution requirements, and essential guardrails for maintaining code quality.

## Repository Overview

Spring Data Valkey is a Spring Data module providing first-class Valkey/Redis integration for Spring applications. It is forked from Spring Data Redis with Valkey-specific enhancements including native Valkey GLIDE driver support.

**Primary Language:** Java 17+
**Build System:** Maven (use `./mvnw` wrapper)
**Framework:** Spring Data 3.5.x, Spring Boot 3.5.x
**Origin:** Forked from Spring Data Redis 3.5.1 — class and package renames follow the pattern `*Redis*` → `*Valkey*`, `*redis*` → `*valkey*` (see [MIGRATION.md](./MIGRATION.md) for full mapping, [DEVELOPER.md](./DEVELOPER.md) for source alignment and detailed build/release info)

**Key Components:**

- `spring-data-valkey/` - Core Spring Data library (connections, operations, repositories, serialization, caching, pub/sub, streams)
- `spring-boot-starter-data-valkey/` - Spring Boot auto-configuration and starter
- `examples/` - Example applications demonstrating various features
- `performance/` - Performance benchmarks
- `docs/` - Astro Starlight documentation site (Node.js)

**Supported Drivers:**

| Driver | Artifact |
|--------|----------|
| Valkey GLIDE (primary) | `io.valkey:valkey-glide` |
| Lettuce | `io.lettuce:lettuce-core` |
| Jedis | `redis.clients:jedis` |

**Driver Notes:**

- Sentinel support is available in Lettuce and Jedis only — GLIDE does not support Sentinel at this time

## Architecture Quick Facts

**Design Pattern:** Spring Data's repository and template abstraction over Valkey connections
**Connection Abstraction:** `ValkeyConnectionFactory` interface with driver-specific implementations (`ValkeyGlideConnectionFactory`, `LettuceConnectionFactory`, `JedisConnectionFactory`)
**Key Abstractions:** `ValkeyTemplate` (operations), `ValkeyRepository` (Spring Data repositories), `ValkeyCache` (Spring Cache), `MessageListenerContainer` (pub/sub)
**Auto-Configuration:** Spring Boot starter auto-configures connections, templates, repositories, and caching via `spring.data.valkey.*` properties
**Serialization:** Pluggable serializers (String, JSON, JDK, XML) — default is JDK serialization for values, String for keys

## Build and Test Rules (Agents)

### Preferred (Make + Maven)

```bash
# Full build with tests (manages Valkey infrastructure automatically)
make test

# Full build including long-running tests
make all-tests

# Run all examples
make examples

# Run performance benchmarks
make performance

# Test infrastructure only
make start       # Start Valkey standalone, sentinel, and cluster instances
make stop        # Stop all instances
make clean       # Clean up configs, pids, and logs
make clobber     # Remove entire work/ directory
```

### Raw Maven Commands

```bash
# Build all modules (skip tests)
./mvnw clean install -DskipTests

# Build all modules with tests (requires running Valkey — use `make start` first)
./mvnw clean install

# Build specific module
./mvnw clean install -pl spring-data-valkey
./mvnw clean install -pl spring-boot-starter-data-valkey

# Run tests for specific module (requires running Valkey)
./mvnw test -pl spring-data-valkey
./mvnw test -pl spring-boot-starter-data-valkey

# Run a single test class
./mvnw test -pl spring-data-valkey -Dtest=ClassName

# Run a single test method
./mvnw test -pl spring-data-valkey -Dtest=ClassName#methodName
```

**Test Results:** Stored in `target/surefire-reports/` within each module directory.

### Running Examples

```bash
# Run all examples (manages infrastructure)
make examples

# Run a single example (requires running Valkey — use `make start` first)
./mvnw -q compile exec:java -pl examples/quickstart
./mvnw -q compile exec:java -pl examples/spring-boot
```

### Running Performance Tests

```bash
# Default performance test with GLIDE (manages infrastructure)
make performance

# Run specific test profiles (requires running Valkey — use `make start` first)
./mvnw -q compile exec:java -pl performance -Dclient=valkeyglide
./mvnw -q compile exec:java -pl performance -Dclient=lettuce
./mvnw -q compile exec:java -pl performance -Dclient=jedis

# Multi-threaded and direct client tests
./mvnw -q compile exec:java@threaded-test -pl performance -Dclient=valkeyglide
./mvnw -q compile exec:java@direct-test -pl performance -Dclient=valkeyglide

# Load test with custom parameters
./mvnw -q compile exec:java@load-test -pl performance -Dclient=valkeyglide -Dthreads=100 -Doperations=200
```

### Documentation (Node.js/Astro)

```bash
cd docs
npm install
npm run dev       # Local dev server at http://localhost:4321/
npm run build     # Production build
```

## Test Infrastructure

Tests require running Valkey instances. The `Makefile` manages this automatically:

- **Standalone:** ports 6379 (master), 6380, 6381 (replicas), 6382 (auth-enabled, password: `foobared`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [valkey-io/spring-data-valkey](https://github.com/valkey-io/spring-data-valkey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
