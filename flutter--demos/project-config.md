---
trigger: always_on
description: This document outlines the best practices and coding standards to be followed
---

# Gemini Code-Gen Best Practices for This Project

This document outlines the best practices and coding standards to be followed
during the development of this Flutter project. Adhering to these guidelines
will ensure the codebase is clean, maintainable, and scalable.

## Architectural Principles

- **DRY (Don’t Repeat Yourself)** – eliminate duplicated logic by extracting
  shared utilities and modules.
- **Separation of Concerns** – each module should handle one distinct
  responsibility.
- **Single Responsibility Principle (SRP)** – every class/module/function/file
  should have exactly one reason to change.
- **Clear Abstractions & Contracts** – expose intent through small, stable
  interfaces and hide implementation details.
- **Low Coupling, High Cohesion** – keep modules self-contained, minimize
  cross-dependencies.
- **Scalability & Statelessness** – design components to scale horizontally and
  prefer stateless services when possible.
- **Observability & Testability** – build in logging, metrics, tracing, and
  ensure components can be unit/integration tested.
- **KISS (Keep It Simple, Sir)** - keep solutions as simple as possible.
- **YAGNI (You're Not Gonna Need It)** – avoid speculative complexity or
  over-engineering.

## Coding Standards

### Linting
This project uses the standard set of lints provided by the `flutter_lints`
package. Ensure that all code adheres to these rules to maintain code quality
and consistency. Run `flutter analyze` frequently to check for linting issues.

### Naming Conventions
- **Files:** Use `snake_case` for file names (e.g., `user_profile.dart`).
- **Classes:** Use `PascalCase` for classes (e.g., `UserProfile`).
- **Methods and Variables:** Use `camelCase` for methods and variables (e.g.,
  `getUserProfile`).
- **Constants:** Use `camelCase` for constants (e.g., `defaultTimeout`).

### Cross-Platform Compatibility
This application targets Android, iOS, web, and macOS. All code must be written
to be platform-agnostic.

-   **Avoid Platform-Specific APIs:** Do not use platform-specific libraries or
    APIs directly (e.g., `dart:io`'s `File` class for UI rendering). When
    platform-specific code is unavoidable, it is abstracted away behind a common
    interface using an adapter pattern, as seen in the `lib/platform` directory.
-   **Use Flutter-Native Solutions:** Prefer Flutter's built-in, cross-platform
    widgets and utilities (e.g., `Image.memory` with byte data for displaying
    images from `image_picker`, which works on all platforms).
-   **Verify Plugin Compatibility:** Before using a new package, ensure it
    supports all target platforms (Android, iOS, web).

### Don't Swallow Errors
- **Don't Swallow Errors** by catching expections, silently filling in required
  but missing values or adding timeouts when something hangs unexpectedly. All
  of those are exceptions that should be thrown so that the errors can be seen,
  root causes can be found and fixes can be applied.
-   **Use Assertions for Invariants:** Use `assert` statements to validate
    assumptions and logical invariants in your code. For example, if a function
    requires a list to be non-empty before proceeding, assert that condition at
    the beginning of the function. This practice turns potential silent failures
    into loud, immediate errors during development, making complex bugs
    significantly easier to track down.

### Null Value Handling
- Prefer using required parameters in constructors and methods when a value is
  not expected to be null.
- When the compiler requires a non-null value and you are certain a value is not
  null at that point, use the `!` (bang) operator. This turns invalid null
  assumptions into runtime exceptions, making them easier to find and fix.
- Avoid providing default values for nullable types simply to satisfy the
  compiler, as this can hide underlying data issues.

### Widget Development
- **`const` Constructors:** Use `const` constructors for widgets whenever
  possible to improve performance by allowing Flutter to cache and reuse widget
  instances.
- **Break Down Large Widgets:** Decompose large widget build methods into
  smaller, more manageable widgets. This improves readability, reusability, and
  performance.

### No Placeholder Code
- We're building production code here, not toys. Avoid placeholder code.

### No Comments for Removed Functionality
- The source is not the place to keep a history of what's changed; it's the
  place to implement the current requirements only. Use version control for
  history.

## Styling and Theming

### Avoid Hardcoded Values
- **Do not** hardcode colors, dimensions, text styles, or other style values
  directly in widgets.
- All centralized style-related code should be consolidated into
  `lib/styles.dart`.
- Create descriptive, `camelCase` constants in a dedicated `lib/styles.dart`
  file for any reusable style values that are not part of the main theme.

### Theme Architecture
- The app uses Material Design 3 with a centralized theme defined in
  `main.dart`.
- All UI components should inherit styles from this central theme. Avoid custom,
  one-off styling for individual widgets.
- Only use per-widget theme or style overrides when a particular widget requires

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flutter/demos](https://github.com/flutter/demos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
