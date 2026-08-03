---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# purchases-hybrid-common — Development Guidelines

This file provides guidance to AI coding agents when working with code in this repository.

## Project Overview

Shared native libraries for RevenueCat's hybrid SDKs (Flutter, React Native, Unity, Capacitor, Cordova). This repository contains iOS (Swift), Android (Kotlin), TypeScript, and JavaScript bridge code that connects platform-native SDKs to hybrid frameworks.

**Related repositories:**
- **iOS SDK**: https://github.com/RevenueCat/purchases-ios — Native iOS SDK that this library wraps. Used by `PurchasesHybridCommon` and `PurchasesHybridCommonUI`.
- **Android SDK**: https://github.com/RevenueCat/purchases-android — Native Android SDK that this library wraps. Used by `hybridcommon` and `hybridcommon-ui`.
- **Flutter SDK**: https://github.com/RevenueCat/purchases-flutter — Uses the iOS and Android hybrid common libraries.
- **React Native SDK**: https://github.com/RevenueCat/react-native-purchases — Uses the iOS and Android hybrid common libraries.
- **Unity SDK**: https://github.com/RevenueCat/purchases-unity — Uses the iOS and Android hybrid common libraries.
- **Capacitor SDK**: https://github.com/RevenueCat/purchases-capacitor — Uses the iOS and Android hybrid common libraries plus TypeScript types.
- **Cordova SDK**: https://github.com/RevenueCat/cordova-plugin-purchases — Uses the iOS and Android hybrid common libraries.
- **KMP SDK**: https://github.com/RevenueCat/purchases-kmp — Kotlin Multiplatform SDK, uses Android hybrid common library.

When implementing features or debugging, check these repos for reference and patterns.

## Important: Public API Stability

**Do NOT introduce breaking changes to the public API.** All hybrid SDKs depend on this library.

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

## Code Structure

```
purchases-hybrid-common/
├── android/                          # Android Kotlin library
│   ├── hybridcommon/                 # Main Kotlin module
│   ├── hybridcommon-ui/              # UI components (PaywallFragment, CustomerCenter)
│   └── api-tests/                    # API integration tests
├── ios/                              # iOS Swift library
│   ├── PurchasesHybridCommon/        # Main Swift module
│   └── PurchasesHybridCommonUI/      # UI components (Paywalls, CustomerCenter)
├── typescript/                       # Shared TypeScript types/interfaces
├── purchases-js-hybrid-mappings/     # JavaScript hybrid mappings for web
├── fastlane/                         # Release automation & CI/CD
└── .circleci/                        # CircleCI configuration
```

## Common Development Commands

### Android

Tool setup: install [mise](https://mise.jdx.dev/) and run `mise install` in the project root (Java, Ruby, Node from `mise.toml`; Yarn via corepack).

```bash
# Build
./gradlew build
./gradlew hybridcommon:build
./gradlew hybridcommon-ui:build

# Tests
./gradlew test

# Enable local purchases-android build
./gradlew enableLocalBuild -PpurchasesPath="~/path/to/purchases-android"

# Disable local build
./gradlew disableLocalBuild

# Code quality
./gradlew detekt
```

### iOS

```bash
# Via CocoaPods
pod install

# Via SwiftPM
swift build

# Code validation
bundle exec fastlane ios pod_lint

# Lint
swiftlint
```

### TypeScript (`@revenuecat/purchases-typescript-internal`)

```bash
yarn build              # TypeScript compilation
yarn build-watch        # Watch mode
yarn lint               # ESLint
yarn extract-api        # API report generation
```

### `@revenuecat/purchases-js-hybrid-mappings`

```bash
npm run build           # Rollup bundling
npm test                # Jest testing
npm run lint            # ESLint
npm run api-test        # API extractor validation
npm run allchecks       # Format + lint + api + test
```

### Cross-Platform (Fastlane)

See [fastlane/README.md](fastlane/README.md) for available lanes. Common commands:

```bash
bundle exec fastlane bump                           # Version bump
bundle exec fastlane check_typescript_api_changes   # API validation
```

**Note:** Deploy lanes should only be run via CI automation, not manually.

## Project Architecture

For iOS and Android, this repository provides two libraries each (core + UI):

### Android (`android/`)

**Core library (`hybridcommon`):**
- **Main Entry**: `hybridcommon/src/main/java/.../common.kt`
- Core functions: `getOfferings()`, `getCurrentOfferingForPlacement()`, `configure()`
- Mappers package handles data transformations
- Used by: Flutter, React Native, Unity, Capacitor, Cordova, KMP

**UI library (`hybridcommon-ui`):**
- Provides `PaywallFragment` and `CustomerCenterFragment` for hybrid SDKs
- Wraps RevenueCatUI components from purchases-android
- Used by: Flutter, React Native, Unity, Capacitor

### iOS (`ios/`)

**Core library (`PurchasesHybridCommon`):**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RevenueCat/purchases-hybrid-common](https://github.com/RevenueCat/purchases-hybrid-common) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
