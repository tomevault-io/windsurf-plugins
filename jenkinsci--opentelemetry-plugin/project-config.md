---
trigger: always_on
description: Agent instructions for the Jenkins OpenTelemetry Plugin.
---

# AGENTS

Agent instructions for the Jenkins OpenTelemetry Plugin.

## Project Overview

**Artifact**: `io.jenkins.plugins:opentelemetry`  
**Type**: Jenkins HPI Plugin (`.hpi` packaging)  
**Language**: Java 21  
**Build Tool**: Maven  

**Purpose**: Monitor and observe Jenkins with OpenTelemetry. Provides distributed tracing for pipeline executions, HTTP requests, metrics collection, and log storage integration with observability backends (Elastic, Jaeger, Grafana, Dynatrace).

**Note**: Version information for dependencies can be found in `pom.xml`.

## Design Principles

Follow these principles when making changes:

- **KISS (Keep It Simple, Stupid)**: Straightforward implementations without over-engineering
- **TDA (Tell, Don't Ask)**: Objects encapsulate behavior and tell what to do rather than exposing state
- **YAGNI (You Aren't Gonna Need It)**: Only implement features that are actually needed
- **DRY (Don't Repeat Yourself)**: Reuse code through well-defined abstractions
- **Structured Programming**: Clear control flow with minimal complexity

## Setup Commands

### Prerequisites
- Java 21 or later
- Maven 3.8.x or later

### Build
```bash
./mvnw clean install
```

### Run with Jenkins
```bash
./mvnw hpi:run
```
This starts Jenkins on http://localhost:8080/jenkins with the plugin loaded.

### Run Demo Environments
```bash
cd demos
make start-elastic  # Elastic Stack (Elasticsearch, Kibana, APM)
make start-grafana  # Grafana Stack (Tempo, Loki)
make start          # Base stack
make stop           # Stop all containers
```

### Code Formatting with Spotless
```bash
# Check code formatting
./mvnw spotless:check

# Apply code formatting
./mvnw spotless:apply
```

### Static Analysis
```bash
# Run SpotBugs analysis
./mvnw spotbugs:check

# Run Error Prone analysis
./mvnw clean verify -P error-prone-check
```

## Code Style Guidelines

### General Java Conventions
- Use `@NonNull` and `@CheckForNull` annotations from FindBugs
- Never return null; use `Optional` for potentially absent values
- Use `@Extension` annotation for Jenkins extension points
- Mark optional extensions with `@Extension(optional = true, dynamicLoadable = YesNoMaybe.YES)`
- Prefer composition over inheritance
- Use interface-based design for extensibility

### Java Best Practices

#### Code Organization
- Keep classes focused on a single responsibility (Single Responsibility Principle)
- Classes should be small: aim for fewer than 200 lines when possible
- Methods should be concise: aim for fewer than 20 lines
- Limit method parameters to 3-4; use parameter objects for more complex cases
- Group related functionality into cohesive packages

#### Naming Conventions
- Classes/Interfaces: Use `PascalCase` (e.g., `MonitoringRunListener`)
- Methods/Variables: Use `camelCase` (e.g., `createSpanBuilder`)
- Constants: Use `UPPER_SNAKE_CASE` (e.g., `MAX_RETRY_ATTEMPTS`)
- Packages: Use lowercase, concatenated words (e.g., `io.jenkins.plugins.opentelemetry`)
- Use meaningful, descriptive names that reveal intent
- Avoid abbreviations unless widely recognized (e.g., `HTTP`, `URL`)
- Boolean variables/methods should indicate true/false (e.g., `isEnabled`, `hasPermission`)

#### Error Handling
- Catch specific exceptions rather than generic `Exception`
- Never catch `Throwable` or `Error` unless absolutely necessary
- Don't ignore exceptions; at minimum, log them
- Use try-with-resources for `AutoCloseable` resources
- Prefer checked exceptions for recoverable conditions
- Document exceptions with `@throws` in Javadoc

#### Null Safety
- Prefer `Optional<T>` over returning null for methods that may not have a value
- Use `@NonNull` and `@CheckForNull` annotations consistently
- Validate method parameters at the beginning of methods
- Avoid passing null as arguments; use overloaded methods instead
- Initialize collections to empty rather than null

#### Collections and Streams
- Prefer immutable collections when possible (e.g., `List.of()`, `Set.of()`)
- Use `Collections.unmodifiableList()` for defensive copies
- Use streams for data processing, but keep pipelines readable (max 3-4 operations)
- Prefer method references over lambdas when possible (e.g., `String::length`)
- Use appropriate collection types: `List` for ordered, `Set` for uniqueness, `Map` for key-value

#### Performance Considerations
- Avoid premature optimization; measure before optimizing
- Use `StringBuilder` for string concatenation in loops
- Cache expensive computations when appropriate
- Be mindful of boxing/unboxing with primitives in collections
- Use lazy initialization for expensive objects when appropriate
- Prefer primitive types over wrapper classes when nullability isn't needed

#### Concurrency
- Minimize mutable shared state
- Use `java.util.concurrent` classes over raw synchronization
- Prefer `ExecutorService` over creating threads directly
- Document thread-safety guarantees in class Javadoc
- Use `volatile` for flags that need visibility guarantees
- Avoid `synchronized` on public methods; use private locks instead

#### Testing
- Write tests for all public APIs
- Use meaningful test method names that describe what is being tested
- Follow Arrange-Act-Assert (AAA) pattern
- Keep tests focused: one logical assertion per test

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jenkinsci/opentelemetry-plugin](https://github.com/jenkinsci/opentelemetry-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
