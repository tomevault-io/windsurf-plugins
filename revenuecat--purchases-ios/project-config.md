---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# purchases-ios — Development Guidelines

This file provides guidance to AI coding agents when working with code in this repository.

## Project Overview

RevenueCat's official iOS SDK for in-app purchases and subscriptions. Supports iOS, macOS, tvOS, watchOS, and visionOS.

**Related repositories:**
- **Android SDK**: https://github.com/RevenueCat/purchases-android
- **Flutter SDK**: https://github.com/RevenueCat/purchases-flutter
- **React Native SDK**: https://github.com/RevenueCat/react-native-purchases
- **Hybrid Common**: https://github.com/RevenueCat/purchases-hybrid-common — shared layer for hybrid SDKs

When implementing features or debugging, check these repos for reference and patterns.

## Important: Public API Stability

**Do NOT introduce breaking changes to the public API.** The SDK is used by thousands of apps.

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

**Do NOT add new `public enum` types to the SDK's consumer-facing surface.** Adding a case to an existing `public enum` is a **source-breaking change** — any consumer with an exhaustive `switch` will fail to compile. Use structs with static constants or other patterns instead when exposing new option sets or categories. Enums marked `@_spi(Internal)` (on the same line as the declaration) are exempt because they are not part of the SDK's consumer-facing surface.

This policy is enforced by the `no_new_public_enums` SwiftLint custom rule defined in `.swiftlint.yml`. Pre-existing public enums are grandfathered in `swiftlint-baseline.json`.

The `Tests/APITesters/` targets run in CI to catch unintended API changes. The `api/*.swiftinterface` files track the public API surface. **If API tests fail, you've likely broken the public API.**

### Objective-C Compatibility

Many core SDK classes are exposed to Objective-C and must stay compatible. Key rules:

- **`NSObject` subclasses** (`Purchases`, `CustomerInfo`, `EntitlementInfo`, `StoreProduct`, `StoreTransaction`, etc.) must remain `@objc`-compatible. Don't add Swift-only types (e.g., generics, `async` without a completion-handler wrapper, Swift enums without `@objc`, default parameter values) to their public API without providing an Obj-C equivalent.
- **`@objc(RC...)` prefixed names** are used for Obj-C class names (e.g., `@objc(RCPurchases)`, `@objc(RCCustomerInfo)`). Don't remove or change these.
- **New public properties/methods** on `@objc`-exposed classes must be marked `@objc` unless there's a deliberate reason to exclude them (document why).
- **Enums** exposed to Obj-C use `@objc` with `Int` raw values. Swift-only enums with associated values or string raw values can't be used from Obj-C.
- **Both Swift and Obj-C API testers exist** in `Tests/APITesters/`. When modifying public API on an `@objc` class, update both `SwiftAPITester` and `ObjcAPITester` targets.
- **Don't break existing Obj-C callers** — if a method is currently callable from Obj-C, it must remain so.

## Common Development Commands

Quick reference for the most common operations:

```bash
swift build                        # Build via SPM
swift test                         # Run unit tests via SPM
tuist generate                     # Generate Tuist workspace (preferred)
swiftlint                          # Run linter
swiftlint --fix                    # Auto-fix lint issues
bundle exec fastlane test_ios      # Run iOS tests via Fastlane
bundle exec fastlane run_api_tests # Verify public API surface
```

For the full set of build, test, Tuist, and Fastlane commands, see:
- **`Contributing/CONTRIBUTING.md`** — environment setup, workflow, style guide
- **`Contributing/DEVELOPMENT.md`** — Tuist workspace generation, targets, troubleshooting
- **`fastlane/README.md`** — complete list of available Fastlane lanes

## Project Architecture

### Module Structure

This is a multi-target Swift project supporting iOS, macOS, tvOS, watchOS, and visionOS:

- **`RevenueCat`** (`Sources/`) — Core SDK: API, business logic, networking, StoreKit abstractions
- **`RevenueCat_CustomEntitlementComputation`** (`CustomEntitlementComputation/` → symlink to `Sources/`) — Same source with `ENABLE_CUSTOM_ENTITLEMENT_COMPUTATION` flag
- **`ReceiptParser`** (`LocalReceiptParsing/` → symlink to `Sources/LocalReceiptParsing/`) — Local receipt parsing library
- **`RevenueCatUI`** (`RevenueCatUI/`) — SwiftUI paywalls and customer center (depends on `RevenueCat`)

Key top-level directories: `Sources/`, `RevenueCatUI/`, `Tests/`, `Examples/`, `Projects/`, `api/`, `fastlane/`, `Tuist/`, `Contributing/`, `scripts/`. Explore the filesystem directly for current subdirectory layout.

### Key Architectural Patterns

#### Core Purchases Module
- **Singleton Pattern**: `Purchases.shared` as the main entry point
- **Delegate Pattern**: `PurchasesDelegate` for event callbacks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RevenueCat/purchases-ios](https://github.com/RevenueCat/purchases-ios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
