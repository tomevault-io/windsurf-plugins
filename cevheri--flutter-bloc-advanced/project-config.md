---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Flutter BLoC Advance Template - a production-ready Flutter application using BLoC pattern for state management. Supports Android, iOS, Web, macOS, Linux, and Windows.

- **Flutter:** 3.44.3 | **Dart SDK:** ^3.12.2 | **FVM** is used for Flutter version management
- **Line width:** 120 characters (configured in `analysis_options.yaml`)

## Common Commands

```bash
# Dependencies
fvm flutter pub get

# Run locally (mock API)
fvm flutter run --target lib/main/main_local.dart

# Run production (real API)
fvm flutter run --target lib/main/main_prod.dart

# Analysis & formatting
fvm dart analyze
fvm dart fix --apply
fvm dart format . --line-length=120

# Tests
fvm flutter test
fvm flutter test test/path/to/specific_test.dart          # single test file
fvm flutter test --coverage                                # with coverage

# Localization code generation (after editing ARB files)
fvm dart run intl_utils:generate

# Build
fvm flutter build apk --release --target lib/main/main_prod.dart
fvm flutter build web --target lib/main/main_prod.dart
```

## Architecture

This project follows a **Feature-First Clean Architecture**. Every development MUST follow this modular approach.

### Project Structure

```
lib/
├── app/                    # Application foundation
│   ├── router/             # Centralized routing configuration
│   └── configuration/      # Global environment setup, logging, storage
├── features/               # Modular business features
│   └── <feature_name>/
│       ├── application/    # BLoCs & Use Cases (Business Logic)
│       ├── data/           # Repositories & Models (Data Layer)
│       ├── navigation/     # Feature-specific route definitions
│       └── presentation/   # Pages & Widgets (UI Layer)
├── shared/                 # Shared foundation across features
│   ├── design_system/      # Theme, typography, color tokens, atoms
│   ├── models/             # Cross-feature models/entities (e.g., UserEntity)
│   ├── utils/              # Generic utility functions
│   └── widgets/            # Reusable components (buttons, forms, etc.)
├── generated/              # Auto-generated localization (do not edit)
├── l10n/                   # ARB translation files
└── main/                   # Entry points (main_local.dart, main_prod.dart)
```

### Developing a Feature

- **Strict Isolation:** Features should not import from other features' internal directories (e.g., `features/user/presentation`).
- **Shared Access:** Cross-feature logic or models MUST be moved to `shared/`.
- **Clean Boundaries:** Each feature module is self-contained. Use Case (Application) layer should only depend on Repository (Data) interfaces.

## BLoC Pattern

Every feature follows: **Event → BLoC → State → UI**
- BLoCs use `Equatable` for state equality.
- Local UI state can be handled within the page, but business logic MUST be in BLoCs.

### `Bloc` vs `Cubit`

- Use **`Cubit`** when every interaction is an atomic, fire-and-forget call and the event stream adds no value (no debouncing, no concurrency policy, no historical event payload needed). Example: `SettingsCubit` (`changeLanguage`, `changeTheme`, `logout`).
- Use **`Bloc`** when events carry meaningful payload, when an `EventTransformer` (debounce, restartable, droppable) is needed, or when the event log itself is valuable for replay/observability. Examples: `UserListBloc` (search debounce + delete drop-concurrent), `UserEditorBloc` (fetch/save lifecycle), `LoginBloc` (multi-step auth flow).
- Default to `Bloc` for any new feature touching network requests or user input streams; reach for `Cubit` only after confirming none of the above apply.

### State Modeling — MAIN RULE

**Default to sealed state hierarchies** that leverage Dart 3's `sealed` modifier and exhaustive `switch` expressions. The compiler enforces handling of every state variant; UIs render via pattern matching.

```dart
// State (real example: lib/features/users/application/user_list_state.dart)
sealed class UserListState extends Equatable {
  const UserListState();
}
final class UserListInitial extends UserListState { /* ... */ }
final class UserListLoading extends UserListState { /* ... */ }
final class UserListLoaded extends UserListState {
  const UserListLoaded({required this.users});
  final List<UserEntity> users;
  @override List<Object?> get props => [users];
}
final class UserListDeleteSuccess extends UserListState { /* ... */ }
final class UserListFailure extends UserListState {
  const UserListFailure({required this.error});
  final String error;
  @override List<Object?> get props => [error];
}

// UI
BlocBuilder<UserListBloc, UserListState>(
  builder: (context, state) => switch (state) {
    UserListInitial() => const SizedBox.shrink(),
    UserListLoading() => const Loading(),
    UserListLoaded(:final users) => UserList(users),
    UserListDeleteSuccess() => const SizedBox.shrink(),
    UserListFailure(:final error) => ErrorBanner(error),
  },
);
```

**Use single-state + status enum only when** one of these is genuinely true:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cevheri/flutter-bloc-advanced](https://github.com/cevheri/flutter-bloc-advanced) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
