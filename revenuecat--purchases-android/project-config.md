---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Common Development Commands

### Building and Testing
```bash
# Build all modules
./gradlew build

# Run unit tests
./gradlew test

# Run only backend integration tests
# (requires BACKEND_INTEGRATION_API_KEY and BACKEND_INTEGRATION_LOAD_SHEDDER_API_KEY in local.properties; tests self-skip if not set)
./gradlew :purchases:testDefaultsDebugUnitTest --tests "com.revenuecat.purchases.backend_integration_tests.*"

# Run unit tests for specific modules (flavor format: {apis}{buildType})
./gradlew :purchases:testDefaultsDebugUnitTest
./gradlew :purchases:testCustomEntitlementComputationDebugUnitTest
./gradlew :ui:revenuecatui:testDefaultsDebugUnitTest

# Run Android instrumentation tests
./gradlew connectedAndroidTest

# Run instrumentation tests for specific modules
./gradlew :purchases:connectedDefaultsDebugAndroidTest
./gradlew :ui:revenuecatui:connectedDefaultsDebugAndroidTest

# Run integration tests (requires device/emulator)
./gradlew :integration-tests:connectedDebugAndroidTest
```

### Code Quality and Analysis
```bash
# Run lint (static code analysis)
./gradlew lint

# Run detekt (static code analysis)
./gradlew detektAll

# Run detekt with auto-correct
./gradlew detektAll --auto-correct

# Create detekt baseline
./gradlew detektAllBaseline

# API compatibility check (using Metalava)
./scripts/api-check.sh

# Generate API signatures (using Metalava)
./scripts/api-dump.sh
```

### UI Screenshot Testing (Paparazzi)
```bash
# Generate/verify snapshots
./gradlew :ui:revenuecatui:recordPaparazziDebug
./gradlew :ui:revenuecatui:verifyPaparazziDebug
```

### Fastlane Commands
```bash
# Setup development environment (links pre-commit hooks)
bundle exec fastlane setup_dev

# Run tests
bundle exec fastlane test

# Run backend integration tests
bundle exec fastlane run_backend_integration_tests
```

## Project Architecture

### Module Structure
This is a multi-module Android project with clear separation of concerns:

- **`:purchases`** - Core SDK module containing main API, business logic, networking, billing abstractions
- **`:ui:revenuecatui`** - Jetpack Compose UI module for paywalls and customer center (min SDK 24, depends on `:purchases`)
- **`:ui:debugview`** - Debug utilities and UI for development (depends on `:purchases`)
- **`:feature:amazon`** - Amazon Appstore integration as separate feature module (depends on `:purchases`)
- **`:bom`** - Bill of Materials for dependency management
- **`:baselineprofile`** - Performance optimization profiles
- **`:integration-tests`** - Integration test suite
- **`:examples/*`** - Sample applications and testers

### Key Architectural Patterns

#### Core Purchases Module
- **Orchestrator Pattern**: `PurchasesOrchestrator` as central coordinator
- **Abstract Factory**: `BillingAbstract` for different store implementations
- **Backend/Cache Layer**: `Backend` for networking, `DeviceCache` for local storage
- **Manager Pattern**: `IdentityManager`, `SubscriberAttributesManager`, `EventsManager`

#### UI Modules
- **MVVM Pattern**: ViewModels with Jetpack Compose UI
- **Main Components**: `PaywallViewModel`, `CustomerCenterViewModel`

### Product Flavors
The `purchases` module has 1 flavor dimension:
- **`apis`**: `defaults` (standard) or `customEntitlementComputation` (custom entitlement computation variant)

Variant names combine the flavor and build type, e.g. `defaultsDebug`, `customEntitlementComputationRelease`.

### API Annotations
- **`@InternalRevenueCatAPI`** - APIs that are public only to be accessible by other modules or hybrid SDKs, not intended for external developer use
- **`@ExperimentalPreviewRevenueCatPurchasesAPI`** - Public APIs for developers that may change before being made stable
- **`@ExperimentalPreviewRevenueCatUIPurchasesAPI`** - Same as above but for the `:ui:revenuecatui` module

## Code Style

- **Imports over inline fully-qualified references**: Always add an `import` statement at the top of the file rather than using a fully-qualified name inline (e.g., write `import foo.Bar` and use `Bar`, not `foo.Bar` inline in the code).

## Testing Framework

### Technologies Used
- **JUnit 4** - Primary testing framework
- **Mockk** - Mocking framework for Kotlin
- **Robolectric** - Android unit testing
- **Espresso** - UI testing
- **AssertJ** - Fluent assertions
- **Paparazzi** - Compose UI screenshot testing
- **Coroutines Test** - Async testing

### Test Structure
- **Unit Tests**: `src/test/` directories
- **Instrumentation Tests**: `src/androidTest/` directories
- **Integration Tests**: `/integration-tests/` module
- **Backend Integration Tests**: `src/test/.../backend_integration_tests/` (requires `BACKEND_INTEGRATION_API_KEY` and `BACKEND_INTEGRATION_LOAD_SHEDDER_API_KEY` in `local.properties`; tests self-skip if not set)

## Development Workflow

### Environment Setup
1. Install [mise](https://mise.jdx.dev/) and run `mise install` in project root to install the pinned JDK (sdkman via `sdk env install` is also supported)
2. Run `bundle exec fastlane setup_dev` to link pre-commit hooks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RevenueCat/purchases-android](https://github.com/RevenueCat/purchases-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
