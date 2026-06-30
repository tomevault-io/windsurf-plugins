---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Blocker is an Android component controller that enables users to manage (enable/disable) Activities, Services, Broadcast Receivers, and Content Providers within installed applications to reduce resource usage and disable unwanted functionality.

**Three Control Mechanisms:**

1. **PackageManager (Root)**: Uses typed librootkotlinx root commands to directly modify component states through PackageManager/ActivityManager APIs, with command-style fallbacks for legacy paths. Changes are persisted in `/data/system/users/0/package_restrictions.xml`. Components are actually disabled, but apps can detect this and potentially re-enable them.

2. **Intent Firewall (IFW)**: Generates XML rules in `/data/system/ifw/` to filter intents at the framework level (Android 4.4.2+). Components appear enabled to the app but cannot start. Requires root access but provides stealthier blocking.

3. **Shizuku/Sui**: Uses Shizuku API for elevated permissions via ADB or root. Non-root mode requires test-only APKs (android:testOnly="true"). See [Shizuku documentation](https://github.com/RikkaApps/Shizuku).

The architecture follows [Now in Android](https://github.com/android/nowinandroid) patterns with strict modularization and official Android architecture guidelines.

## Build Configuration

- **Compile SDK**: 36
- **Target SDK**: 36
- **Min SDK**: 23
- **JDK**: 21
- **Gradle**: 9.1.0
- **Kotlin**: 2.2.0

## Build Commands

### Build Variants

Two product flavors with multiple build types:
- **foss**: Open-source build without Firebase/GMS (F-Droid version)
- **market**: Google Play build with Firebase Analytics and Crashlytics

Build types: `debug`, `release`, `benchmarkRelease`, `nonMinifiedRelease`

### Common Tasks

```bash
# Build debug APK (foss flavor)
./gradlew assembleFossDebug

# Build release APK (market flavor)
./gradlew assembleMarketRelease

# Build release bundle for Play Store
./gradlew bundleMarketRelease

# Install debug build on connected device
./gradlew installFossDebug

# Clean build
./gradlew clean

# Build all variants
./gradlew assemble
```

### Testing

```bash
# Run all unit tests
./gradlew test

# Run tests for specific module
./gradlew :core:data:test
./gradlew :feature:applist:testFossDebugUnitTest

# Run screenshot tests (verify against baselines)
./gradlew verifyRoborazziFossDebug

# Record/update screenshot baselines (MUST run on Linux)
./gradlew recordRoborazziFossDebug

# Run instrumented tests on connected device
./gradlew connectedFossDebugAndroidTest

# Run specific instrumented test
./gradlew :app-compose:connectedFossDebugAndroidTest
```

### Code Quality

```bash
# Run lint checks
./gradlew lint

# Run all checks (tests + lint)
./gradlew check

# Format code with Spotless
./gradlew spotlessApply

# Check code formatting
./gradlew spotlessCheck

# Generate JaCoCo coverage report
./gradlew createCombinedCoverageReport

# Update dependency baselines
./gradlew dependencyGuard
```

### Other Useful Commands

```bash
# Generate module dependency graphs (Mermaid in READMEs)
./gradlew graphUpdate

# List all available tasks
./gradlew tasks

# Build with baseline profile generation (release only)
./gradlew :app-compose:assembleMarketRelease
```

## Architecture

### Module Structure

**Core Modules (`core/`)** - Shared infrastructure and domain logic:
- `analytics`: Analytics event tracking abstraction
- `common`: Shared utilities, extensions, and base classes
- `component-controller`: Component control implementations (Root/IFW/Shizuku)
- `data`: Repository implementations orchestrating local/remote data sources
- `database`: Room database schemas and DAOs
- `datastore`: Proto DataStore for user preferences
- `datastore-proto`: Protocol buffer definitions
- `designsystem`: Material 3 design system and reusable UI components
- `domain`: Business logic use cases
- `git`: Git operations for syncing rule repositories
- `ifw-api`: Intent Firewall XML generation and parsing
- `model`: Data models and domain entities
- `network`: Retrofit-based API clients for remote rule fetching
- `provider`: Content provider utilities
- `rule`: Rule matching engine for component patterns
- `ui`: Shared UI components used across features
- `testing`: Test utilities, fakes, and test doubles

**Feature Modules (`feature/`)** - Screen-level features:
- `applist`: Application list with filtering and sorting
- `appdetail`: Application detail showing components (activities, services, receivers, providers)
- `generalrule`: General rule management and synchronization
- `ruledetail`: Detailed rule viewing and editing
- `search`: Global search with regex support
- `settings`: User preferences and controller configuration

**App Module (`app-compose/`)**: Main application module with navigation setup, dependency injection configuration, and app-level components.

**Sync Module (`sync/work`)**: Background synchronization using WorkManager.

**Supporting Modules**:
- `build-logic`: Custom Gradle convention plugins
- `benchmarks`: Baseline profile generation
- `lint`: Custom lint rules
- `ui-test-hilt-manifest`: Test-specific Hilt configuration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lihenggui/blocker](https://github.com/lihenggui/blocker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
