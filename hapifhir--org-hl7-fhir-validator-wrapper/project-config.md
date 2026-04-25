---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the FHIR Validator Wrapper - a Kotlin Multiplatform project that provides CLI, Desktop GUI, and Web-based validation server for FHIR resources. It combines:
- **JVM backend** using Ktor for REST API and validation engine (wraps HAPI FHIR validation library)
- **JS frontend** using React with Context API (Kotlin/JS) and Material-UI
- **Common shared code** for models and constants

A publicly hosted instance: https://validator.fhir.org/

## Build & Development Commands

### Building
```bash
./gradlew build              # Full build (JVM + JS)
./gradlew jvmJar             # Build JVM JAR only
./gradlew jsJar              # Build JS frontend only
./gradlew clean              # Clean build artifacts
```

### Running
```bash
./gradlew run                # Run local server (localhost:8080)
java -jar build/libs/validator-wrapper-jvm-*.jar -startServer   # Full-stack server
java -jar build/libs/validator-wrapper-jvm-*.jar -gui           # Desktop app mode
```

### Testing
```bash
./gradlew test               # Run all tests
./gradlew jvmTest            # JVM tests only
./gradlew jsTest             # JS tests only (uses Karma + Chrome Headless)
./gradlew jsBrowserTest      # JS browser tests
```

To run a single test class:
```bash
./gradlew jvmTest --tests "ClassName"
./gradlew jvmTest --tests "controller.validation.*"
```

### Version Management
Version is managed via semantic versioning plugin. Check `version.properties` for current version.
```bash
./gradlew printVersion       # Print current version
```

## Architecture Overview

### Kotlin Multiplatform Structure

The codebase uses Kotlin's `expect`/`actual` pattern to share code across platforms:

```
src/
├── commonMain/              # Shared models, constants, expected declarations
│   ├── kotlin/
│   │   ├── model/          # Data models (ValidationRequest, ValidationResponse, etc.)
│   │   └── constants/      # API endpoints, FHIR formats, MIME types
│   └── resources/          # Static content (index.html, Swagger UI)
├── jvmMain/                # JVM server implementation
│   ├── kotlin/
│   │   ├── Server.kt       # Main entry point (3 execution modes)
│   │   ├── Module.kt       # Ktor configuration
│   │   ├── controller/     # REST route handlers (by feature)
│   │   ├── api/            # Dependency injection (Koin)
│   │   └── utils/          # Caching, file handling
├── jsMain/                 # React frontend with Context API
│   ├── kotlin/
│   │   ├── Main.kt         # Bootstrap entry & context provider setup
│   │   ├── App.kt          # Root React component (consumes contexts)
│   │   ├── api/            # HTTP client (Ktor client)
│   │   ├── context/        # React Context providers & state
│   │   │   ├── AppScreenContext.kt      # Screen navigation state
│   │   │   ├── LocalizationContext.kt   # i18n & language state
│   │   │   └── ValidationContext.kt     # Validation state (files, config, IGs)
│   │   ├── ui/components/  # React UI hierarchy (context consumers)
│   │   ├── utils/          # Helper functions
│   │   └── css/            # Styling utilities
```

### JVM Backend Architecture

**Entry Point & Execution Modes** (Server.kt:39):
- `-startServer`: Full-stack hosted server (default)
- `-gui`: Desktop app with embedded Chromium window
- No args: Defaults to server mode

**Dependency Injection** (Koin):
- Controllers and services are injected via `ControllersInjection.kt` and `ApiInjection.kt`
- Enables testability and loose coupling

**Feature-Based Controllers**:
Each feature has a dedicated controller directory with:
- Interface definition
- Implementation class
- Module.kt with Ktor route definitions

Example structure:
```
controller/
├── validation/
│   ├── ValidationController.kt          # Interface
│   ├── ValidationControllerImpl.kt      # Implementation
│   └── ValidationModule.kt              # Routes: post("/validate")
├── ig/                                  # Implementation Guides
├── version/                             # Version info
└── terminology/                         # Terminology server validation
```

**Key Backend Classes**:
- `ValidationServiceFactory`: Creates validation service instances (wraps HAPI FHIR validator)
- `SessionCacheFactory`: Manages session-based caching with Guava
- `PackageCacheDownloaderRunnable`: Preloads package cache on startup (controlled by `PRELOAD_CACHE` env var)

### JS Frontend Architecture

**React Context State Management**:

The frontend uses React's Context API for centralized state management with three context providers:

**Context Provider Hierarchy:**
```kotlin
// Main.kt - Provider nesting order
AppScreenProvider {           // Outermost - Screen navigation
  LocalizationProvider {      // Middle - i18n & translations
    ValidationProvider {      // Innermost - Validation state (most complex)
      App { ... }
    }
  }
}
```

**The Three Contexts:**

1. **AppScreenContext** (`context/AppScreenContext.kt`):
   - **State**: `appScreen: AppScreen` (VALIDATOR | SETTINGS enum)
   - **Callback**: `setAppScreen: (AppScreen) -> Unit`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hapifhir) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
