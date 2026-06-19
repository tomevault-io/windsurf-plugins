---
trigger: always_on
description: Flutter Intents is a Flutter plugin that bridges iOS App Intents and Android AppFunctions frameworks, enabling Flutter apps to integrate with Siri, Shortcuts, Spotlight, and AI agents (Gemini etc.).
---

# Flutter Intents - AI Codebase Guide

## Project Overview

Flutter Intents is a Flutter plugin that bridges iOS App Intents and Android AppFunctions frameworks, enabling Flutter apps to integrate with Siri, Shortcuts, Spotlight, and AI agents (Gemini etc.).

## Package Structure

```
packages/
├── app_intents_annotations/  # Dart annotations for Intent/Entity definitions
├── app_intents/              # Flutter plugin (Platform Interface + Method Channel)
└── app_intents_codegen/      # build_runner code generator
ios-spm/
└── AppIntentsBridge/         # Swift Package for iOS native bridge
app/                          # Example Flutter application
docs/
├── architecture.md           # System architecture and design rationale
├── packages.md               # Package responsibilities and dependencies
└── usage.md                  # User guide and integration instructions
```

## Key Design Decisions

| Decision | Choice |
|----------|--------|
| iOS Minimum | **iOS 17** |
| AppShortcutsProvider | **Supported** |
| Handler Registration | **Auto-registration** (code-generated) |
| Localization | **String Catalog** (iOS standard) |
| Error Handling | **Both** (iOS standard + custom) |
| Entity Images | **URL + Asset + SF Symbol** |
| Intent Execution (iOS) | **URL Scheme** (due to Flutter engine timing) |
| Intent Execution (Android) | **MethodChannel** (in-process, no URL scheme needed) |
| Deep Linking | **app_links** package |
| Android Minimum | **API 36** (Android 16, for AppFunctions) |
| Android AppFunctions | **Jetpack `androidx.appfunctions` 1.0.0-alpha09** |
| Cross-Process Storage (iOS) | **App Group UserDefaults** (explicit configuration required) |
| WWDC26 New APIs | **Opt-in, default OFF** (`#if APP_INTENTS_WWDC26`, dual-branch generation) |

## Implementation Status

### Completed
- **WWDC26 experimental codegen (opt-in, #52 Intent execution control)**
  - `ExperimentalFeatures` config (`lib/src/experimental/experimental_features.dart`): master switch + per-feature set; default OFF reproduces stable output byte-for-byte
  - CLI `generate_swift`: `--experimental-wwdc26` (master) + `--experimental=long-running,app-schema` (per-feature)
  - `@IntentSpec(longRunning:, cancellable:, executionTargets:)` + `IntentExecutionTarget` enum
  - `SwiftGenerator` emits **two struct variants per intent**: WWDC26 form in `#if APP_INTENTS_WWDC26`, stable form in `#else` (so released-SDK builds without the flag still compile)
  - Verified: dual-branch `swiftc -typecheck` green against Xcode 27 beta iOS 27 SDK (with and without `-D APP_INTENTS_WWDC26`)
  - See "WWDC26 Experimental Code Generation" under Code Conventions for the SDK-verified API facts
- **WWDC26 App Schema (#49) + semantic indexing (#50)**
  - `@EntitySpec(schema:)` / `@IntentSpec(schema:)` (e.g. `'messages.message'`) → dual-branch `@AppEntity(schema: .messages.message)` / `@AppIntent(schema: .messages.setMessageReadStatus)` gated by the `app-schema` experimental feature (iOS 27). The entity struct, its query and extensions all move to iOS 27 in the `#if` branch.
  - `@EntityProperty(title:, indexingKey:)` → Swift `@Property(...)`; `indexingKey: 'contentDescription'` emits `@Property(indexingKey: \.contentDescription)` for semantic indexing. **Normal feature (no experimental flag)** gated at `@available(iOS 18.4, *)`.
  - Entities that expose `@Property` get an explicit initializer (the `@Property`/`EntityProperty` wrapper has **no `init(wrappedValue:)`**), with defaults so the role-only construction in the generated query keeps compiling.
  - Verified: dual-branch `swiftc -typecheck` green for schema × indexed × enumerable × `@Property` combinations (stable @ iOS 18.4, experimental @ iOS 27).
- **WWDC26 enum schema (#49) + entity ownership (#55)**
  - `@EnumSpec(schema:)` → dual-branch `@AppEnum(schema: .messages.messageType)` (gated by the `app-schema` feature, iOS 27). The `@AppEnum(schema:)` macro is lenient like the entity/intent variants.
  - `@EntitySpec(ownership:)` (`EntityOwnershipState.unknown/.shared/.public`) → an additive `OwnershipProvidingEntity` conformance extension (`var ownership: EntityOwnership { .shared }`) in its own `#if APP_INTENTS_WWDC26` block (no `#else` — without the flag the entity simply isn't ownership-aware). Gated by the new `ownership` experimental feature (iOS 27).
  - Verified: dual-branch `swiftc -typecheck` green (stable @ iOS 17, experimental @ iOS 27).
- **WWDC26 richer parameter types (#53 — `Duration` + `PersonName` + `EntityCollection` + `@UnionValue`)**
  - **`Duration`**: a Dart `Duration` `@IntentParam` generates a real duration parameter. **By default (and in the `#else` fallback)** it maps to `Measurement<UnitDuration>` (stable SDK; Shortcuts shows a duration picker). With the `rich-types` experimental feature it upgrades to the **native iOS 27 `Duration`** inside `#if APP_INTENTS_WWDC26` (dual-branch). Both branches pre-serialize to a normalized `<field>Micros: Int` local (native via `.components`; `Measurement` via `.converted(to: .seconds).value`); Dart deserializes with `Duration(microseconds:)`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [touyou/flutter_intents](https://github.com/touyou/flutter_intents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
