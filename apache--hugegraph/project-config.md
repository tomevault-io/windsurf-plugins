---
trigger: always_on
description: This file provides guidance to an AI coding tool when working with code in this repository.
---

# AGENTS.md

This file provides guidance to an AI coding tool when working with code in this repository.

## Project Overview

hugegraph-commons is a shared utility module for Apache HugeGraph and its peripheral components. It provides core infrastructure components (locks, config, events, iterators, REST client, RPC framework) to simplify development across the HugeGraph ecosystem.

**Technology Stack**:
- Java 8+ (compiler source/target: 1.8)
- Apache Maven 3.5+
- Apache Commons Configuration2 for config management
- OkHttp 4.10.0 for REST client (hugegraph-common)
- Sofa-RPC 5.7.6 for RPC framework (hugegraph-rpc)
- JUnit 4.13.1 and Mockito 4.1.0 for testing

## Architecture

### Two-Module Structure

This is a Maven multi-module project with 2 main modules:

1. **hugegraph-common**: Core utilities library
   - Lock implementations (atomic, key, row, lock groups)
   - Configuration system with type-safe options
   - Event hub for async notifications
   - Iterator utilities (map, filter, flat-map, batch)
   - RESTful client (OkHttp-based)
   - Utilities (perf analysis, version checking, collections, logging)
   - License management

2. **hugegraph-rpc**: RPC communication framework
   - Sofa-RPC based client/server implementation
   - Consumer and provider configuration
   - Service registration and discovery
   - **Depends on hugegraph-common**

### Key Design Patterns

1. **Type-Safe Configuration System**: `HugeConfig` + `OptionSpace` pattern
   - Config options defined as typed `ConfigOption` objects
   - Supports both `.properties` and `.yaml` files
   - Options organized in `OptionSpace` groups for validation
   - Security checks on load

2. **Lock Hierarchy**: Multiple lock implementations for different use cases
   - `AtomicLock`: Basic atomic locking
   - `KeyLock`: Lock by specific key
   - `RowLock`: Row-level locking for table-like structures
   - `LockGroup`: Manage multiple related locks
   - `LockManager`: Central lock coordination

3. **Event System**: Async event notification
   - `EventHub`: Central event dispatcher
   - `EventListener`: Typed event handlers
   - Thread-safe event publishing

4. **Iterator Composition**: Chainable iterator wrappers
   - `MapperIterator`, `FilterIterator`, `LimitIterator`
   - `FlatMapperIterator` for nested iteration
   - `BatchMapperIterator` for batch processing
   - All extend `ExtendableIterator` base

5. **RPC Architecture**: Sofa-RPC abstraction layer
   - `RpcServer`: Service provider side
   - `RpcClientProvider`: Service consumer side
   - `RpcProviderConfig`/`RpcConsumerConfig`: Configuration wrappers
   - Supports multiple protocols (bolt, rest, grpc)

## Build & Development Commands

### Prerequisites
```bash
# Verify Java version (8+ required)
java -version

# Verify Maven version (3.5+ required)
mvn -version
```

### Build Commands

```bash
# Clean build without tests (fastest)
mvn clean install -DskipTests

# Build with tests enabled
mvn clean install

# Build specific module only
mvn clean install -pl hugegraph-common -DskipTests
mvn clean install -pl hugegraph-rpc -am -DskipTests  # -am includes dependencies

# Compile with warnings visible
mvn clean compile -Dmaven.javadoc.skip=true
```

**Note**: Tests are skipped by default via `<skipCommonsTests>true</skipCommonsTests>` in pom.xml. To run tests, override with `-DskipCommonsTests=false`.

### Testing

```bash
# Run all tests (override default skip)
mvn test -DskipCommonsTests=false

# Run tests for specific module
mvn test -pl hugegraph-common -DskipCommonsTests=false
mvn test -pl hugegraph-rpc -am -DskipCommonsTests=false

# Run single test class
mvn test -pl hugegraph-common -Dtest=HugeConfigTest -DskipCommonsTests=false

# Run test suite (includes all unit tests)
mvn test -pl hugegraph-common -Dtest=UnitTestSuite -DskipCommonsTests=false
```

### Code Quality

```bash
# License header check (Apache RAT)
mvn apache-rat:check

# Checkstyle validation
mvn checkstyle:check

# Both checks run automatically during validate phase
mvn validate
```

### Code Coverage

```bash
# Generate JaCoCo coverage report
mvn clean test -DskipCommonsTests=false
# Report: target/jacoco/index.html
```

## Important File Locations

### Source Code Structure
- hugegraph-common sources: `hugegraph-common/src/main/java/org/apache/hugegraph/`
  - `concurrent/`: Lock implementations
  - `config/`: Configuration system (HugeConfig, OptionSpace, ConfigOption)
  - `event/`: Event hub and listeners
  - `iterator/`: Iterator utilities
  - `rest/`: REST client implementation
  - `util/`: Various utilities (collections, logging, version, etc.)
  - `perf/`: Performance measurement (PerfUtil, Stopwatch)
  - `license/`: License management

- hugegraph-rpc sources: `hugegraph-rpc/src/main/java/org/apache/hugegraph/`
  - `rpc/`: RPC server and client implementations
  - `config/`: RPC-specific config options

### Test Structure
- Unit tests: `hugegraph-{module}/src/test/java/org/apache/hugegraph/unit/`
- Test suites: `UnitTestSuite.java` lists all test classes
- Test utilities: `hugegraph-common/src/main/java/org/apache/hugegraph/testutil/`
  - `Whitebox`: Reflection utilities for testing private members
  - `Assert`: Enhanced assertion utilities

### Configuration Files

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/hugegraph](https://github.com/apache/hugegraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
