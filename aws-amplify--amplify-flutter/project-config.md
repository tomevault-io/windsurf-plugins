---
trigger: always_on
description: git submodule update --init --recursive
---

# Project Info

## Setup

```bash
git submodule update --init --recursive
dart pub global activate -spath packages/aft
aft bootstrap
```

## Linting

Changes MUST pass:
```
flutter analyze
```

### Auto format:
```
dart pub get # crucial to run this!
dart fix --apply .
dart format  .
```

## Project Coding Conventions & Patterns Summary

### Project Overview
- **Language**: Dart (Flutter + pure Dart)
- **Architecture**: Monorepo with ~25+ packages under `packages/`
- **Tooling**: Custom `aft` (Amplify Flutter Tool) for bootstrapping, formatting, analysis, testing
- Root `pubspec.yaml` - Flutter/Dart monorepo configuration with multiple packages

---

### 1. File & Directory Naming
- **snake_case** for all Dart files: `auth_plugin_impl.dart`, `state_machine.dart`, `amplify_exception.dart`
- **snake_case** for directories: `amplify_core`, `aws_common`, `amplify_auth_cognito_dart`
- Package names follow pattern: `amplify_<category>` for Flutter, `amplify_<category>_dart` for pure Dart
- Generated files use `.g.dart` suffix (json_serializable)
- Platform-conditional files use suffixes: `globals.flutter.dart`, `globals.dart.dart`, `initial_parameters_stub.dart` / `initial_parameters_html.dart`

### 2. License Header (Required on ALL files)
```dart
// Copyright Amazon.com, Inc. or its affiliates. All Rights Reserved.
// SPDX-License-Identifier: Apache-2.0
```
Enforced via `tool/license.sh` and CI checks.

### 3. Code Style & Lint Rules
- Every `analysis_options.yaml` MUST include a shared lint profile from `packages/amplify_lints/`:
  - **Library packages**: `include: package:amplify_lints/library.yaml`
  - **Example/app packages**: `include: package:amplify_lints/app.yaml`
- The profiles handle all lint rules, strict mode settings, and enforced conventions. Packages may add local overrides (e.g., excluding `.g.dart` files) but must not replace the base include.

### 4. Documentation Patterns
- **Dartdoc template/macro system** used extensively:
  ```dart
  /// {@template amplify_core.amplify_exception}
  /// Description here.
  /// {@endtemplate}
  ```
  Then referenced elsewhere:
  ```dart
  /// {@macro amplify_core.amplify_exception}
  ```
- **Category tags** for dartdoc categorization: `/// {@category Auth}`
- All public APIs have doc comments (enforced by `public_member_api_docs`)
- Code examples in docs use `<?code-excerpt>` syntax for verified code excerpts

### 5. Architecture Patterns

#### 5a. Category/Plugin Architecture
- **Categories**: Abstract interfaces (`AuthCategory`, `StorageCategory`, etc.) define the public API
- **Plugins**: Concrete implementations (`AmplifyAuthCognitoDart`) implement category interfaces
- **Plugin Key pattern**: Static `pluginKey` constant for type-safe plugin retrieval:
  ```dart
  static const AuthPluginKey<AmplifyAuthCognitoDart> pluginKey = _AmplifyAuthCognitoDartPluginKey();
  ```

#### 5b. State Machine Pattern (Core architectural pattern)
- Extensive use of typed state machines for complex flows (auth, sign-in, sign-out, etc.)
- Hierarchy: `StateMachineManager` → `StateMachine` → `StateMachineEvent` / `StateMachineState`
- States are `sealed` classes with named constructors for each variant:
  ```dart
  sealed class SignInState extends AuthState<SignInStateType> {
    const factory SignInState.notStarted() = SignInNotStarted;
    const factory SignInState.success(AuthUser user) = SignInSuccess;
    const factory SignInState.failure({...}) = SignInFailure;
  }
  ```
- State types expressed as enums: `SignInStateType { notStarted, initiating, challenge, success, failure }`
- `SuccessState` and `ErrorState` mixins for terminal states
- `EventCompleter<E, S>` for async event tracking with `accepted`/`completed` futures
- Events have `checkPrecondition()` method for guard conditions

#### 5c. Dependency Injection
- `DependencyManager` (service locator pattern) with `addBuilder()`, `addInstance()`, `get()`, `getOrCreate()`
- `Token<T>` for type-safe dependency keys
- `_ScopedDependencyManager` for hierarchical scope

#### 5d. Dart/Flutter Split
- Pure Dart packages (`_dart` suffix) for cross-platform logic
- Flutter packages wrap Dart packages for platform integration
- Conditional imports for web vs. VM: `if (dart.library.js_interop)`

### 6. Type System Patterns

#### 6a. Mixins as Core Building Blocks
- `AWSEquatable<T>` — Value equality via `props` list (like Equatable package)
- `AWSDebuggable` — Safe `toString()` via `runtimeTypeName` (prevents runtime reflection)
- `AWSSerializable<T>` — `toJson()` contract
- `AmplifyLoggerMixin` — Logging integration
- Classes commonly compose multiple mixins:
  ```dart
  class AmplifyOutputs with AWSEquatable<AmplifyOutputs>, AWSSerializable, AWSDebuggable { ... }
  ```

#### 6b. Sealed Classes & Pattern Matching
- Heavy use of Dart 3 sealed classes for states and results:
  ```dart
  sealed class AWSResult<V, E extends Exception> { ... }
  final class AWSSuccessResult<V, E> extends AWSResult<V, E> { ... }
  final class AWSErrorResult<V, E> extends AWSResult<V, E> { ... }
  ```
- `final class` for concrete implementations (prevent extension)
- `base class` / `base mixin` for state machine types (controlled hierarchy)
- Dart 3 pattern matching (`switch` expressions, `case` patterns) used throughout

#### 6c. Immutability

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws-amplify/amplify-flutter](https://github.com/aws-amplify/amplify-flutter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
