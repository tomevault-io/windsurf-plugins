---
trigger: always_on
description: This document provides guidance for AI coding agents working with the Java Operator SDK codebase.
---

# AI Agents Guide for Java Operator SDK

This document provides guidance for AI coding agents working with the Java Operator SDK codebase.

## Project Overview

Java Operator SDK is a production-ready framework for building Kubernetes Operators in Java. It provides:
- A controller runtime for reconciliation loops
- Support for dependent resources and workflows
- Testing utilities for operator development
- Integration with Fabric8 Kubernetes Client

**Key Technologies:**
- Java 17 (compilation target), validated in CI against Java 17, 21, and 25
- Maven for build management
- Fabric8 Kubernetes Client for K8s API access
- JUnit 6 (via `org.junit:junit-bom`) for testing
- GitHub Actions for CI/CD

## Project Structure

### Core Modules

```
java-operator-sdk/
├── operator-framework-core/        # Core reconciliation engine and API
├── operator-framework/             # Main operator framework implementation
├── operator-framework-junit/      # Testing utilities and extensions
├── operator-framework-bom/         # Bill of Materials for dependency management
├── micrometer-support/             # Metrics integration
├── open-telemetry-support/         # Distributed tracing support
├── caffeine-bounded-cache-support/ # Caching implementation
├── bootstrapper-maven-plugin/      # Maven plugin for bootstrapping
└── test-index-processor/           # Test utilities for annotation processing
```

### Key Packages

- `io.javaoperatorsdk.operator.api.reconciler` - Core reconciler interfaces and annotations
- `io.javaoperatorsdk.operator.processing` - Event processing and workflow engine
- `io.javaoperatorsdk.operator.processing.dependent` - Dependent resource management
- `io.javaoperatorsdk.operator.api.config` - Configuration interfaces
- `io.javaoperatorsdk.operator.junit` - Testing support classes

## Working Effectively

### Build Commands

```bash
# Full build with tests
./mvnw clean install

# Build without tests
./mvnw clean install -DskipTests

# Parallel build (uses 1 thread per CPU core)
./mvnw -T1C clean install

# Check code formatting
./mvnw spotless:check

# Apply code formatting
./mvnw spotless:apply

# Check license headers
./mvnw -N license:check
```

### Test Execution

```bash
# Run unit tests only
./mvnw test

# Run integration tests
./mvnw verify -Pintegration-tests

# Run specific test class
./mvnw test -Dtest=ClassName

# Run specific test method
./mvnw test -Dtest=ClassName#methodName
```

## Code Conventions

### Code Style

- Formatting: The project uses Spotless with Google Java Format
- License Headers: All source files must have Apache 2.0 license headers
- Line Length: 100 characters maximum
- Indentation: 2 spaces (no tabs)
- Prefer `var` to avoid type declarations, except for very short type names like `int`, `long`, `String` etc.
- Always use proper imports for classes instead of fully qualified class references. Import classes at the top of the file and use simple class names throughout the code, only using fully qualified names when absolutely necessary to avoid naming collisions.
- Add unit and/or integration tests for new functionality whenever reasonably possible
- Avoid excessive logging, only add logs to critical parts. Avoid both logging errors and throwing exceptions at the same time. Throwing the error is enough it is logged already somewhere else.
- Do not add comments to the code, except in case of very long or complex logic.

### Naming Conventions

- **Reconcilers:** End with `Reconciler` (e.g., `MyResourceReconciler`)
- **Dependent Resources:** End with `DependentResource` (e.g., `ConfigMapDependentResource`)
- **Test Classes:** End with `Test` for unit tests, `IT` for integration tests, `E2E` for end-to-end testing.
- **Custom Resources:** Typically structured as `{Name}Spec`, `{Name}Status`, `{Name}` (the CR class)

### API Design

- Use builder patterns for complex configurations
- Prefer immutable objects where possible
- Use annotations for declarative configuration (`@ControllerConfiguration`, `@KubernetesDependent`, etc.)
- Follow fluent API design for DSLs

## Testing Guidelines

### Unit Tests

- Use JUnit 6
- Mock Kubernetes API interactions using Fabric8's mock server or Mockito; or service layer directly
- Test reconciliation logic in isolation
- Place in `src/test/java`

### Integration Tests

- Use `LocallyRunOperatorExtension` or `OperatorExtension` from `operator-framework-junit`
- Test against real Kubernetes API (typically via test cluster like minikube or kind)
- Suffix with `IT` (e.g., `MyReconcilerIT`)
- Located in `src/test/java`

### Test Resources

- Kubernetes manifests in `src/test/resources`

## Common Patterns

### Reconciler Implementation

Reconcilers implement the `Reconciler<T>` interface:

```java
@ControllerConfiguration
public class MyReconciler implements Reconciler<MyCustomResource> {

  @Override
  public UpdateControl<MyCustomResource> reconcile(
      MyCustomResource resource, Context<MyCustomResource> context) {
    // Reconciliation logic
    return UpdateControl.noUpdate();
  }
}
```

### Dependent Resources

Dependent resources use the `DependentResource` interface or extend base classes:

```java
@KubernetesDependent

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [operator-framework/java-operator-sdk](https://github.com/operator-framework/java-operator-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
