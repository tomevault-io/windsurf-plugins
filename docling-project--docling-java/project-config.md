---
trigger: always_on
description: `docling-java` is a multi-module Java library providing a Java API for [Docling](https://github.com/docling-project), an IBM Research project for AI-based document processing (PDF, DOCX, PPTX, images, audio, etc.).
---

# Docling Java – Copilot Instructions

## Project Overview

`docling-java` is a multi-module Java library providing a Java API for [Docling](https://github.com/docling-project), an IBM Research project for AI-based document processing (PDF, DOCX, PPTX, images, audio, etc.).

The project is published to Maven Central under the `ai.docling` group ID.

## Repository Structure

```
docling-java/
├── buildSrc/                         # Convention plugins (Gradle Kotlin DSL)
│   └── src/main/kotlin/
│       ├── docling-shared.gradle.kts          # group/version resolution
│       ├── docling-java-shared.gradle.kts     # java-library + jacoco + JUnit 5 test config
│       ├── docling-lombok.gradle.kts          # Lombok setup
│       └── docling-release.gradle.kts         # maven-publish setup
├── gradle/libs.versions.toml         # Version catalog (single source of truth for deps)
├── gradle.properties                 # Gradle settings (java.version=17, parallel, caching)
├── settings.gradle.kts               # Module declarations
├── docling-core/                     # Core DoclingDocument model (no runtime deps)
├── docling-serve/
│   ├── docling-serve-api/            # Framework-agnostic API interfaces + request/response types
│   └── docling-serve-client/         # Reference HTTP client (Java HttpClient + Jackson)
├── docling-testcontainers/           # Testcontainers module for Docling Serve
├── docling-testing/
│   └── docling-version-tests/        # Quarkus/Picocli CLI for version-compatibility testing
├── docs/                             # MkDocs documentation site
├── test-report-aggregation/          # Aggregated JaCoCo + JUnit reports
└── .github/
    ├── project.yml                   # release.current-version (source of truth for version)
    └── workflows/                    # CI: build.yml, release.yml, docs.yml, version-tests.yml
```

### Module ↔ Gradle project name mapping

| Directory path | Gradle project name |
|---|---|
| `docling-core` | `:docling-core` |
| `docling-serve/docling-serve-api` | `:docling-serve-api` |
| `docling-serve/docling-serve-client` | `:docling-serve-client` |
| `docling-testcontainers` | `:docling-testcontainers` |
| `docling-testing/docling-version-tests` | `:docling-version-tests` |

## Build System

- **Gradle** with **Kotlin DSL** (`build.gradle.kts`, `settings.gradle.kts`).
- **Java 17** is the baseline; CI also tests Java 21 and 25.
- Convention plugins in `buildSrc/` keep module `build.gradle.kts` files minimal.
- All dependency versions live in `gradle/libs.versions.toml` (version catalog).
- Project version is read from `.github/project.yml` (`release.current-version`) by `docling-shared.gradle.kts`.

## Common Build Commands

```bash
# Build and test a single module (recommended during development)
./gradlew :docling-serve-api:clean :docling-serve-api:build
./gradlew :docling-serve-client:clean :docling-serve-client:build
./gradlew :docling-testcontainers:clean :docling-testcontainers:build
./gradlew :docling-core:clean :docling-core:build

# Run tests for a specific module
./gradlew :docling-serve-api:test
./gradlew :docling-serve-client:test

# Specify a different Java version
./gradlew -Pjava.version=21 :docling-serve-client:build

# Generate aggregated test report
./gradlew :test-report-aggregation:check

# Build the documentation site
./gradlew :docs:build

# Run the version-compatibility CLI (dev mode, requires Docker)
./gradlew :docling-version-tests:quarkusDev
```

> **Note:** Tests in `docling-serve-client` that use WireMock also start a `DoclingServeContainer` via Testcontainers and therefore require a running Docker daemon. Tests in `docling-testcontainers` that use Testcontainers likewise require Docker, while tests in `docling-serve-api` do not use WireMock and can run without Docker.

## Java Code Conventions

### Lombok

All model/value types use **Lombok** annotations. The standard pattern for immutable value objects is:

```java
@lombok.Builder(toBuilder = true)
@lombok.Getter
@lombok.ToString
@lombok.extern.jackson.Jacksonized   // for Jackson deserialization via builder
public class MyType {
  @Nullable
  private String optionalField;
  private String requiredField;
}
```

- Use `@lombok.Singular` on collection fields for builder singular-adder methods.
- A `lombok.config` file exists in module source roots; do not remove it.

### Nullability

Use **JSpecify** annotations for nullability:

```java
import org.jspecify.annotations.Nullable;

@Nullable
private String mayBeNull;   // field/param/return that may be null
// no annotation = non-null by default
```

### Jackson (dual Jackson 2 & 3 support)

The project supports **both Jackson 2** (`com.fasterxml.jackson`) and **Jackson 3** (`tools.jackson`). When annotating models:

- Use `com.fasterxml.jackson.annotation.*` for Jackson 2/3-compatible annotations (`@JsonProperty`, `@JsonInclude`, `@JsonSetter`, etc.).
- Use `@tools.jackson.databind.annotation.JsonDeserialize` for Jackson 3-specific deserializer wiring.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [docling-project/docling-java](https://github.com/docling-project/docling-java) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
