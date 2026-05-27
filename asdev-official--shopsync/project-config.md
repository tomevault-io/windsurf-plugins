---
trigger: always_on
description: ShopSync is a Flutter app for collaborative shopping list management with Firebase backend. It supports **three platforms**: phone (`lib/main.dart`) and WearOS (`lib/wear/wear_main.dart`) with **separate build flavors**. The web version does not have a flavor, however it is actively maintained with the rest of the app.
---

# ShopSync AI Coding Agent Instructions

## Project Overview

ShopSync is a Flutter app for collaborative shopping list management with Firebase backend. It supports **three platforms**: phone (`lib/main.dart`) and WearOS (`lib/wear/wear_main.dart`) with **separate build flavors**. The web version does not have a flavor, however it is actively maintained with the rest of the app.

## Architecture & Key Patterns

### Dual-Target Architecture

- **Phone app**: `flutter run --release --flavor phone -d <device>`
- **WearOS app**: `flutter run --release --flavor wear -d <device> --target lib/wear/wear_main.dart`
- Flavors defined in `android/app/build.gradle` with dimension "platform"
- Separate entry points but shared services, models, and Firebase configuration

### Firebase Data Structure

```
list_groups/ (top-level grouping)
  └─ {groupId}
     ├─ name, createdBy, members[], position, isExpanded, listIds[]
     └─ lists/ (subcollection references by listIds[])

lists/ (shopping lists)
  └─ {listId}
     ├─ name, createdBy, members[], color, place, isRecycleBin
     ├─ items/ (subcollection)
     │  └─ {itemId}: name, checked, quantity, categoryId, deleted, etc.
     └─ categories/ (subcollection)
        └─ {categoryId}: name, iconIdentifier, order
```

**Critical**: Lists can be grouped but also exist standalone. Use `ListGroupsService` for group operations, direct Firestore queries for list/item CRUD.

### State Management

- **No Provider/ChangeNotifier** for global state (commented out in codebase)
- Uses `StreamBuilder<QuerySnapshot>` and `StreamBuilder<DocumentSnapshot>` for real-time Firebase sync
- `FutureBuilder` for one-time async operations (e.g., checking update status)
- Local state via `setState()` and `StatefulWidget`

### Services Layer Pattern

All services are static utility classes (no singletons/instances):

- `ListGroupsService` - group CRUD, reordering, list-group associations
- `CategoriesService` - per-list categories with order management
- `GoogleAuthService` - handles Credential Manager (Android) + web auth flows
- `SmartSuggestionsService` - on-device ML for item suggestions (trains from user history)
- `ConnectivityService` - monitors network state, shows offline dialog
- `HomeWidgetService` - Android home widget updates via `home_widget` package

**Error handling**: All service methods wrap errors with `Sentry.captureException()` including contextual hints.

## Development Workflows

### Building & Running

```bash
# Phone flavor (default target)
flutter run --release --flavor phone -d emulator-5554

# WearOS flavor (custom target required)
flutter run --release --flavor wear -d emulator-5556 --target lib/wear/wear_main.dart

# CI checks (what GitHub Actions runs)
flutter analyze --no-fatal-infos
flutter pub get
```

**Linting**: Uses `flutter_lints` with `use_build_context_synchronously: ignore` (see `analysis_options.yaml`).

### Localization

- ARB files in `lib/l10n/`, generated via `l10n.yaml` config
- Run `./extract_strings.sh` to auto-generate `app_en.arb` from code (extracts `Text()`, `title:`, `return` strings)
- Currently translations are not being added inside app. Hardcode strings as normal.

### Release Process

- **Phone**: CD workflow builds `--flavor phone --target=lib/main.dart` → `app-phone-release.aab`
- **WearOS**: Separate CD workflow for wear builds
- Version code format: `XXYYYYYYY` where XX=platform (30=phone,40=wear), YYYYYYY=versionCode.
- Requires `key.properties` (keystore config) and `sentry.properties` (debug symbols upload)

## Code Conventions

### Localization

**CRITICAL**: All user-facing strings in UI elements MUST be localized.

- **Never** use hardcoded strings in UI components (Text, labels, titles, messages, etc.)
- **Always** use `AppLocalizations.of(context)!` to access localized strings
- **Always** add new strings to `lib/l10n/app_en.arb` when creating UI elements
- String keys should be camelCase and descriptive (e.g., `aiFeatures`, `enableSmartSuggestions`)
- Include context in key names when needed (e.g., `aiFeaturesEnabled` vs `aiFeaturesDisabledMessage`)

**Example**:

```dart
// ❌ WRONG - Hardcoded string
Text('AI Features')

// ✅ CORRECT - Localized string
Text(AppLocalizations.of(context)!.aiFeatures)
```

**Adding new strings**:

1. Add the string to `lib/l10n/app_en.arb`:
   ```json
   "aiFeatures": "AI Features",
   "enableSmartSuggestions": "Enable Smart Suggestions"
   ```
2. Use it in code:
   ```dart
   final l10n = AppLocalizations.of(context)!;
   Text(l10n.aiFeatures)
   ```

**Note**: Currently, translations are not being added to other locale files. Only `app_en.arb` needs to be updated with English strings.

### File Organization

- `lib/screens/` - full-page UI (e.g., `home.dart`, `list_view.dart`)
- `lib/widgets/` - reusable components (e.g., `expandable_list_group_widget.dart`)
- `lib/services/` - business logic & Firebase interactions
- `lib/wear/screens/` - WearOS-specific UI (circular layouts, rotary support)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ASDev-Official/shopsync](https://github.com/ASDev-Official/shopsync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
