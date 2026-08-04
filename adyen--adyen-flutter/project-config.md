---
trigger: always_on
description: This document provides rules and context for LLMs working on this project.
---

# AGENTS.md - LLM Guidelines for adyen-flutter

This document provides rules and context for LLMs working on this project.

## Project Overview

**adyen-flutter** is a Flutter plugin for integrating Adyen Checkout into mobile applications. It
wraps the native Adyen Android and iOS SDKs, exposing them to Flutter via platform channels.

**Supported Platforms:** Android and iOS **only**. Do not implement or consider other platforms (
web, desktop, etc.).

## Project Structure

```
adyen-flutter/
├── lib/                              # Dart code
│   ├── adyen_checkout.dart           # Public API entry point
│   └── src/
│       ├── common/                   # Shared models and utilities
│       │   └── model/                # Dart model classes
│       ├── components/               # Individual payment components
│       ├── drop_in/                  # Drop-in integration
│       ├── generated/                # Pigeon-generated Dart code (DO NOT EDIT)
│       ├── logging/                  # Logging utilities
│       └── util/                     # Utility classes
├── pigeons/
│   └── platform_api.dart             # Pigeon API definitions (DTOs + interfaces)
├── android/
│   └── src/main/kotlin/com/adyen/checkout/flutter/
│       ├── generated/                # Pigeon-generated Kotlin code (DO NOT EDIT)
│       ├── dropIn/                   # Drop-in implementation
│       ├── components/               # Component implementations
│       └── utils/                    # Utility classes
├── ios/adyen_checkout/
│   └── Sources/adyen_checkout/
│       └── generated/                # Pigeon-generated Swift code (DO NOT EDIT)
└── example/                          # Example Flutter app
```

## Planning Flutter Features

- **[Write the plan]** Store implementation plans in the appropriate plan folder.
- **[Problem statement + non-goals]** Start the plan with a short problem statement and explicit
  non-goals/out-of-scope.
- **[Public API sketch first]** Define the intended Dart-facing API early (types + a short usage
  snippet). Keep it Flutter-idiomatic and minimal.
- **[Ownership + data flow]** Decide and document what happens in Flutter vs native.
- **[Acceptance criteria]** Add a small checklist of observable behaviors, including
  backwards-compat expectations ("when omitted, existing behavior is unchanged").
- **[Clarify unknowns]** Ask questions when requirements are unclear instead of making assumptions.
- **[Phase-based delivery]** Break work into phases that can be completed and validated
  independently. A good default for cross-platform features is:
    - **[Flutter models]** Public Dart API models.
    - **[Pigeon DTOs + codegen]** Define DTOs in `pigeons/platform_api.dart`, then generate.
    - **[Dart DTO mapping]** Map public models to DTOs in `lib/src/util/dto_mapper.dart`.
    - **[Android mapping]** Map DTOs to Android SDK types.
    - **[iOS mapping]** Map DTOs to iOS SDK types.
    - **[Finalization]** Exports, tests, example usage, docs, changelog.
- **[Keep the public API clean]** Prefer a Flutter-idiomatic abstraction (simple Dart models) and do
  the complex mapping internally. Avoid exposing native SDK configuration graphs directly.
- **[Backwards compatibility]** New fields should be optional and default to current behavior when
  omitted.
- **[DTO structure]** When adding platform configuration, design DTOs to match the native SDK
  structure the platform code needs, minimizing transformation on native.
- **[Named parameters for DTOs]** Prefer named parameters when constructing DTOs (especially
  Pigeon-generated DTOs) and any constructors with many fields. This improves readability and avoids
  breakage when constructor signatures change.
- **[Value/format decisions]** Decide early where conversions happen (e.g., `Color` to hex string,
  `double` to `int` rounding) and keep that logic in one place.
- **[Validation strategy]** Validate obvious invalid values on the Flutter side where feasible (for
  clearer errors), while keeping defaults/omitted values delegated to the native SDK.
- **[Testing + manual verification]** Plan both unit tests (Flutter mapping) and manual verification
  steps (Android + iOS), especially for UI changes.
- **[Risks + mitigations]** List the top risks (platform differences, invalid values, SDK
  limitations) and how you’ll validate/mitigate.
- **[Done definition]** End the plan with “done when” items (codegen committed, tests passing,
  example updated, docs/changelog updated).

## Idiomatic Flutter Practices

### General

- **Effective Dart**: Follow [Effective Dart](https://dart.dev/effective-dart) guidelines.
- **Lints**: Respect the `flutter_lints` rules configured in `analysis_options.yaml`.
- **Asynchrony**: Use `async`/`await` for better readability over `.then()`.
- **Null Safety**: Avoid force unwrapping (`!`). Handle nulls gracefully.

### Plugin Development

- **Public API**: Expose clean, idiomatic Dart models to the user. Do not expose DTOs or internal
  implementation details.
- **Error Handling**: Translate native errors into meaningful Dart exceptions.
- **Platform Views**: Ensure proper lifecycle management (creation, updates, disposal) to prevent
  memory leaks.

## Platform Communication with Pigeon


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Adyen/adyen-flutter](https://github.com/Adyen/adyen-flutter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
