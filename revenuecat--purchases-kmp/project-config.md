---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# purchases-kmp — Development Guidelines

This file provides guidance to AI coding agents when working with code in this repository.

## Project Overview

RevenueCat's official Kotlin Multiplatform (KMP) SDK for in-app purchases and subscriptions. Enables code sharing between iOS and Android platforms using Kotlin Multiplatform.

**Related repositories:**
- **iOS SDK**: https://github.com/RevenueCat/purchases-ios
- **Android SDK**: https://github.com/RevenueCat/purchases-android
- **Hybrid Common**: https://github.com/RevenueCat/purchases-hybrid-common — Native bridge layer

When implementing features or debugging, check these repos for reference and patterns.

## Important: Public API Stability

**Do NOT introduce breaking changes to the public API.** The SDK is used by production apps.

**Safe changes:**
- Adding new optional parameters to existing methods
- Adding new classes, methods, or properties
- Bug fixes that don't change method signatures
- Internal implementation changes

**Requires explicit approval:**
- Removing or renaming public classes/methods/properties
- Changing method signatures (parameter types, required params)
- Changing return types
- Modifying behavior in ways that break existing integrations

The project uses **kotlinx.binary-compatibility-validator** for API stability checking.

## Code Structure

```
purchases-kmp/
├── core/                     # Main SDK entry point (Purchases class, configuration)
│   └── src/
│       ├── commonMain/       # Shared Kotlin code
│       ├── androidMain/      # Android-specific implementations
│       └── iosMain/          # iOS-specific implementations (Swift interop)
├── models/                   # Shared data models and domain objects
├── mappings/                 # Platform-specific mappings
├── revenuecatui/             # Jetpack Compose UI components for paywalls
├── either/                   # Either/Result type implementations
├── result/                   # Result wrapper types
├── build-logic/              # Custom Gradle build convention plugins
├── composeApp/               # KMP sample application
├── apiTester/                # API testing application
├── iosApp/                   # iOS demo application
└── fastlane/                 # Release automation
```

## Common Development Commands

```bash
# Build all modules
./gradlew build

# Run all tests
./gradlew test

# Run Detekt linting
./gradlew detektAll

# Generate documentation
./gradlew dokkatooGenerate

# Assemble without tests
./gradlew assemble

# Publish locally
./gradlew publishToMavenLocal
```

### Platform-Specific Tests

```bash
# Common KMP tests
./gradlew commonTest

# Android unit tests
./gradlew androidUnitTest

# iOS tests (requires macOS)
./gradlew iosTest
```

## Project Architecture

### Main Entry Point: `Purchases` Class
**Location**: `core/src/commonMain/kotlin/com/revenuecat/purchases/kmp/Purchases.kt`

- **Singleton Pattern**: `Purchases.sharedInstance` (set via `configure()`)
- **Initialization**: `Purchases.configure(PurchasesConfiguration)` — must be called early in app lifecycle
- **Configuration Builder**: `PurchasesConfiguration.Builder(apiKey)` — fluent builder pattern

### Configuration Options
- `apiKey` (required) — RevenueCat API key
- `appUserId` (optional) — User identifier
- `purchasesAreCompletedBy` — Whether RevenueCat or app completes purchases
- `storeKitVersion` — iOS StoreKit 1 vs 2 selection
- `showInAppMessagesAutomatically` — Control of native in-app messages
- `verificationMode` — Entitlement verification settings

### Module Dependencies

```
:revenuecatui --> :mappings, :core, :models
:core --> :models, :mappings
:mappings --> :models
```

### Source Set Organization

Each module follows standard KMP structure:
```
module/src/
├── commonMain/       # Shared Kotlin code (Android + iOS)
├── commonTest/       # Shared tests
├── androidMain/      # Android-specific implementations
├── androidUnitTest/  # Android unit tests
├── iosMain/          # iOS-specific implementations
└── iosTest/          # iOS tests
```

## Constraints / Support Policy

| Platform | Minimum Version |
|----------|-----------------|
| Kotlin | 2.3.20 |
| Android | minSdk 23, compileSdk 35 |
| iOS (Core) | 13.0+ |
| iOS (UI) | 15.0+ |
| Java | 8+ |

Don't raise minimum versions unless explicitly required and justified.

## Testing

```bash
# All tests
./gradlew test

# Detekt linting
./gradlew detektAll

# API compatibility check
# (uses kotlinx.binary-compatibility-validator)
```

## Development Workflow

1. Install [mise](https://mise.jdx.dev/) and run `mise install` in project root to install the pinned JDK
2. Build: `./gradlew build`
3. Make changes following KMP source set conventions
4. Run tests: `./gradlew test`
5. Run linting: `./gradlew detektAll`
6. Verify documentation: `./gradlew dokkatooGenerate`

## Pull Request Labels

When creating a pull request, **always add one of these labels** to categorize the change:

| Label | When to Use |
|-------|-------------|
| `pr:feat` | New user-facing features or enhancements |
| `pr:fix` | Bug fixes |
| `pr:other` | Internal changes, refactors, CI, docs, or anything that shouldn't trigger a release |

## When the Task is Ambiguous


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RevenueCat/purchases-kmp](https://github.com/RevenueCat/purchases-kmp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
