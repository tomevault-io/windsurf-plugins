---
trigger: always_on
description: MoodLog is a mood tracking and journaling Flutter app with the following key features:
---

# MoodLog Flutter Project - Gemini Guidelines

## Project Overview

MoodLog is a mood tracking and journaling Flutter app with the following key features:

- Local-first architecture (no login required)
- Mood tracking with AI-powered insights
- Private diary entries stored locally
- Statistics and analytics

## Project Structure

- **State Management**: Provider
- **Routing**: go_router
- **Database**: Drift (SQLite) for local storage
- **AI Integration**: Firebase AI (Gemini)
- **Analytics**: Firebase Analytics
- **Ads**: Google Mobile Ads

### Directory Guidelines: `services` vs. `utils`

To maintain a clean and scalable architecture, it's important to differentiate between `services`
and `utils`.

#### `core/services`

This directory contains classes that provide core functionalities or manage interactions with
external systems. They often have a lifecycle and maintain state.

- **Role**: Manages a specific app-wide service (e.g., logging, flavor management, ads).
- **State**: Often stateful and may be managed as singletons.
- **Lifecycle**: Typically initialized once when the app starts.
- **Examples**: `LoggingService`, `FlavorService`, `AdmobService`.

#### `core/utils`

This directory contains stateless helper functions or classes that perform common, reusable tasks.

- **Role**: Provides simple, reusable tools and helper functions (e.g., formatters, converters,
  keyboard management).
- **State**: Should be stateless.
- **Lifecycle**: No specific lifecycle; used on-demand.
- **Examples**: `ErrorHandler`, `KeyboardUtils`, `StringListConverter`.

## Architecture

### MVVM + Clean Architecture

```
lib/
├── core/           # Common utilities, constants
├── data/           # Repositories, data sources, models
├── domain/         # Entities, repository interfaces, use cases
└── presentation/   # Presentation Layer
    ├── screens/      # UI (View)
    ├── providers/    # State Management (ViewModel)
    ├── widgets/      # Common Widgets
    ├── theme/        # App themes
    └── ...
```

### File Patterns

- **Views (Screens)**: `lib/presentation/screens/*/*_view.dart`
- **ViewModels**: `lib/presentation/providers/*_view_model.dart` or
  `lib/presentation/screens/*/*_view_model.dart`
- **Widgets**: `lib/presentation/widgets/` or `lib/presentation/screens/*/widgets/`
- **Models**: `lib/data/models/`
- **Entities**: `lib/domain/entities/`

#### View File Structure (`part`/`part of`)

To ensure consistency and separation of concerns within the presentation layer, all view files
should follow the `part`/`part of` convention:

- **`*_view.dart`**: This file should contain only one `StatelessWidget` (the main screen widget)
  which is responsible for providing the `ViewModel` to the widget tree.
- **`*_content.dart`**: This file, marked with `part of '*_view.dart'`, contains the `_Content`
  widget which builds the actual UI of the screen.

This structure separates the DI setup from the UI implementation, making the code cleaner and easier
to navigate.

## Key Architectural Decisions (v1.0.28+)

### Local-First Architecture

**No Authentication Required**:

- Firebase Auth has been removed
- Users are created locally on first app launch
- All data is stored locally (Drift + SharedPreferences)

**User Management**:

- `LocalUser` entity with: userId (UUID), nickname, profileImagePath, createdAt
- Stored in SharedPreferences
- No email or external account linking

**Onboarding Flow**:

1. First launch → Show onboarding screens
2. User sets nickname and AI personality
3. Local user automatically created
4. App ready to use

**Data Storage**:

- Journal entries: Drift (SQLite)
- User info & settings: SharedPreferences
- No cloud sync (future feature)

## Development Conventions

### Internationalization

- **Supported Languages**: Korean (default), English, Japanese
- **Translation Files**: `lib/core/l10n/app_{language_code}.arb`
- Only edit `.arb` files; `.dart` files are auto-generated
- Base language is Korean (ko)

### Code Generation

```bash
# Generate localization files
flutter gen-l10n

# Generate freezed/json_serializable code
dart run build_runner build --delete-conflicting-outputs
```

### Common Commands

```bash
flutter analyze          # Lint check
flutter test            # Run tests
flutter build apk       # Build Android APK
```

### Code Style

- Use `withValues(alpha: ..)` instead of deprecated `withOpacity`
- Follow MVVM pattern: View → ViewModel → UseCase → Repository
- Use freezed for immutable entities and models
- Use Provider for state management
- **Avoid unnecessary comments**: Write self-explanatory code. Only add comments for complex logic, important business rules, or non-obvious implementations

### Freezed Usage Convention

When creating a class using the `freezed` package, it must be defined as an `abstract class` with a `mixin`. Using a plain class can lead to unexpected build errors.

```dart
// ✅ Good
@freezed
abstract class User with _$User {
  const factory User({
    required String name,
    required int age,
  }) = _User;
}

// ❌ Avoid
@freezed
class User with _$User {
  const factory User({
    required String name,
    required int age,
  }) = _User;
}
```

### Provider Usage Convention


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/log-mind-365) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
