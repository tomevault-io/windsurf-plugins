---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Behavioral

1. Don’t assume. Don’t hide confusion. Surface tradeoffs.
2. Minimum code that solves the problem. Limit speculative additions.
3. Touch only what you must, clean up only your own mess -- but do suggest additional related fixes.
4. Define success criteria. Loop until verified.

## Project Overview

This is **jackson-databind**, the general-purpose data-binding functionality and tree-model for Jackson Data Processor. It builds on the Streaming API (jackson-core) and uses jackson-annotations for configuration. This is the 3.x branch (Jackson 3.0+) which requires JDK 17+ and uses the `tools.jackson` package namespace (2.x used `com.fasterxml.jackson`).

**Key characteristics:**
- ~790 test files with comprehensive test coverage
- Thread-safe mapper instances (as of Jackson 3.0)
- Maven-based build system with Maven wrapper (`./mvnw`)
- Supports multiple JDK versions (17, 21, 25) with special test profiles

## Common Development Commands

### Building and Testing

```bash
# Full build with tests
./mvnw clean verify

# Build without tests (faster)
./mvnw clean install -DskipTests

# Run tests only
./mvnw test

# Run a specific test class
./mvnw test -Dtest=ClassName

# Run a specific test method
./mvnw test -Dtest=ClassName#methodName

# Generate test report
./create-test-report.sh  # runs: mvn surefire-report:report

# Verify Android SDK compatibility
./mvnw animal-sniffer:check

# JDK 21+ test sources: the `java21` profile auto-activates on JDK 21+,
# so no flag is normally needed (`-Pjava21` just forces it on)
```

### Code Quality

```bash
# Run with ErrorProne static analysis
./mvnw verify -Perrorprone

# Generate code coverage report
./mvnw test jacoco:report
# Report will be in target/site/jacoco/

# Check dependencies
./mvnw dependency:tree
```

## Code Architecture

### Core Components Hierarchy

1. **ObjectMapper** (`ObjectMapper.java`): The main entry point for all Jackson databind operations
   - Thread-safe and fully immutable (as of 3.0)
   - Uses builder pattern for construction (`JsonMapper.builder()` for JSON)
   - Contains caches for serializers/deserializers

2. **Serialization Path** (`ser/` package):
   - `SerializerFactory` / `BeanSerializerFactory`: Creates serializers
   - `ValueSerializer`: Base class for all serializers
   - `BeanSerializer`: Handles POJO serialization
   - `BeanPropertyWriter`: Writes individual bean properties
   - `SerializationContext`: Context for serialization process

3. **Deserialization Path** (`deser/` package):
   - `DeserializerFactory` / `BeanDeserializerFactory`: Creates deserializers
   - `ValueDeserializer`: Base class for all deserializers
   - `BeanDeserializer` (in `deser/bean/`) / `BeanDeserializerBuilder`: Handles POJO deserialization
   - `SettableBeanProperty`: Represents a settable bean property
   - `DeserializationContext`: Context for deserialization process

4. **Type System** (`type/` package):
   - `JavaType` (in root `databind` package): Represents Java types with full generic information
   - `TypeFactory`: Creates JavaType instances
   - Critical for handling generics correctly

5. **Introspection** (`introspect/` package):
   - `AnnotatedClass`, `AnnotatedMethod`, `AnnotatedField`: Represents annotated members
   - `AnnotationIntrospector` (in root `databind` package): Processes annotations to configure behavior
   - Handles reflection and metadata extraction

6. **Configuration**:
   - `MapperConfig` (in `cfg/`): Base configuration
   - `SerializationConfig` / `DeserializationConfig` (in root `databind` package): Specific configurations
   - `MapperFeature`, `SerializationFeature`, `DeserializationFeature` (in root `databind` package): Feature flags
   - `PackageVersion` (in `cfg/`): Generated file containing version information

7. **Tree Model**:
   - `JsonNode` (in root `databind` package): Abstract base for all node types
   - `ObjectNode`, `ArrayNode`, `StringNode`, etc. (in `node/` package): Concrete node types.
     Note: 2.x's `TextNode` is named `StringNode` in 3.x
   - Alternative to POJO binding for dynamic structures

### Package Organization

- `tools.jackson.databind` - Core classes (ObjectMapper, configs, features)
- `tools.jackson.databind.ser` - Serialization infrastructure
- `tools.jackson.databind.deser` - Deserialization infrastructure
- `tools.jackson.databind.type` - Type system and TypeFactory
- `tools.jackson.databind.introspect` - Reflection and metadata
- `tools.jackson.databind.node` - Tree model (JsonNode hierarchy)
- `tools.jackson.databind.annotation` - Databind-specific annotations
- `tools.jackson.databind.json` - JSON-specific mapper (JsonMapper)
- `tools.jackson.databind.jsontype` - Polymorphic type handling
- `tools.jackson.databind.jsonFormatVisitors` - Schema generation visitors
- `tools.jackson.databind.exc` - Exception types
- `tools.jackson.databind.util` - Utility classes
- `tools.jackson.databind.module` - Module system
- `tools.jackson.databind.ext` - External type integrations

### Important Design Patterns

1. **Builder Pattern**: ObjectMapper uses immutable builder pattern (3.x change)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FasterXML/jackson-databind](https://github.com/FasterXML/jackson-databind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
