---
trigger: always_on
description: Nostrino is a Nostr SDK for Kotlin, providing a multiplatform implementation of the Nostr protocol. The project implements various NIPs (Nostr Implementation Possibilities) and is used by Block/CashApp for Nostr-based features. Nostrino supports JVM, iOS, and Linux platforms.
---

# AGENTS.md - LLM Contribution Guide for Nostrino

## Project Overview

Nostrino is a Nostr SDK for Kotlin, providing a multiplatform implementation of the Nostr protocol. The project implements various NIPs (Nostr Implementation Possibilities) and is used by Block/CashApp for Nostr-based features. Nostrino supports JVM, iOS, and Linux platforms.

## Technology Stack

- **Language**: Kotlin (Multiplatform)
- **Build System**: Gradle with Kotlin DSL
- **Java Version**: JVM 11
- **Test Framework**: JUnit 5 (Jupiter) + Kotest
- **Platforms**: JVM, iOS (x64, ARM64, Simulator ARM64), Linux x64
- **Key Dependencies**:
  - OkHttp (HTTP client for JVM)
  - Moshi (JSON)
  - acinq secp256k1 (Cryptographic curves)
  - Kotlinx Coroutines (Asynchronous programming)
  - OkIO (I/O operations)
  - Kotest (Testing and assertions)
  - Turbine (Flow testing)

## Build & Development Commands

### Prerequisites
This project uses [Hermit](https://cashapp.github.io/hermit/) for consistent tooling. Activate it with:
```bash
. ./bin/activate-hermit
```

### Essential Commands

**IMPORTANT**: This project uses Hermit. Always use `bin/gradle` instead of `gradlew` or `gradle`.

```bash
# Build the entire project (includes tests)
bin/gradle build

# Run tests only
bin/gradle test

# Clean build
bin/gradle clean build

# Check for API breaking changes (binary compatibility)
bin/gradle apiCheck

# Generate new API dumps after API changes
bin/gradle apiDump

# Check for dependency updates
bin/gradle dependencyUpdates -Drevision=release

# Update version catalog
bin/gradle versionCatalogUpdate

# Generate API documentation
bin/gradle dokkaHtml
```

## Project Structure

```
nostrino/
├── lib/                    # Main SDK library (multiplatform)
│   └── src/
│       ├── commonMain/kotlin/   # Platform-agnostic code
│       └── jvmMain/kotlin/      # JVM-specific code
├── lib-test/              # Testing utilities and integration tests
│   └── src/
│       ├── main/kotlin/        # Test helpers and Arb generators
│       └── test/kotlin/        # Integration tests
├── buildSrc/              # Build configuration
└── gradle/                # Gradle wrapper and version catalog
    └── libs.versions.toml # Dependency versions
```

## Code Conventions

### General Style
- **Indentation**: 2 spaces (NOT tabs)
- **Line Length**: Maximum 120 characters
- **Line Endings**: LF (Unix-style)
- **Charset**: UTF-8
- **Final Newline**: Always insert
- **Copyright Headers**: Required for all new files. Always use the current year (2025) in the copyright year field, not the project start year (2023)

### Kotlin-Specific
- **Imports**: No wildcard imports (use explicit imports)
  - `ij_kotlin_name_count_to_use_star_import = 2147483647`
  - `ij_kotlin_name_count_to_use_star_import_for_members = 2147483647`
- **Naming**: Follow standard Kotlin conventions
- **Null Safety**: Leverage Kotlin's null safety features
- **Function Syntax**: Prefer expression syntax over block body. Avoid early returns when possible
  - ✅ `fun add(a: Int, b: Int): Int = a + b`
  - ✅ `fun isValid(x: Int): Boolean = x > 0`
  - ❌ `fun add(a: Int, b: Int): Int { return a + b }`
- **Parameters**: Wrap on every item with proper formatting
  - Method parameters: new line after `(`, right paren on new line
  - Call parameters: new line after `(`, right paren on new line
- **Code Style**: Follow Kotlin official code style

### Testing

#### Testing Philosophy: Sociable Unit Testing

Nostrino follows the **Sociable Unit Testing** approach for comprehensive, resilient test coverage.

**Core Principle**: Write unit tests for all classes, but let them connect through to real dependencies until hitting the system boundary.

- ✅ **Real object graphs**: Classes use their actual dependencies (services, stores, validators)
- ✅ **Fake external services**: Only external APIs and services are faked at system boundaries
- ✅ **Fast execution**: Still runs quickly despite using real components
- ✅ **Less brittle**: No need to update mocks when internal implementations change

**System Boundaries** (what gets faked):
- External relay servers (use FakeRelay for testing)
- External HTTP endpoints
- Nostr relay connections

**Benefits over traditional mock-based testing**:
- Tests behavior, not implementation details
- Resilient to refactoring
- Real integration confidence
- Simpler test setup

#### Test Framework & Conventions

- **Framework**: JUnit 5 (Jupiter) + Kotest - configured in root `build.gradle.kts`
- **Test Style**: ALWAYS use Kotest StringSpec (test names in strings)
- **Assertions**: Use Kotest matchers (`shouldBe`, `shouldBeEqual`, `shouldContainExactly`, etc.) - NOT JUnit assertions
- **Test Naming**: Test class suffix is `Test` (e.g., `RelaySetTest.kt`)
- **Test Names**: Use string literals for readable test names (StringSpec style)
- **Integration Tests**: Use Docker containers for relay testing (nostr-rs-relay)
- **Mocking**: Use fake implementations (e.g., `FakeRelay`) for external system boundaries, not internal dependencies

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [block/nostrino](https://github.com/block/nostrino) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
