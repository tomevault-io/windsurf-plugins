---
trigger: always_on
description: This file provides guidance to Claude Code when working with this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

SKKUBUS (스꾸버스) — a Flutter mobile app for Sungkyunkwan University providing real-time shuttle bus tracking, campus maps, building info, and campus services. Targets Android and iOS from a single Dart codebase.

**Stack:** Flutter 3.x, Dart >=3.7.0 (Dart 3 pattern matching used throughout), GetX state management

## Architecture

### Feature-First Structure

- `lib/core/` — shared infrastructure used across features
- `lib/features/` — self-contained feature modules

### core/ vs features/ Boundary Rules

- **core/**: Code shared by 2+ features OR app-wide infrastructure (API layer, routes, services, types, utils, shared widgets)
- **features/**: Self-contained modules. Each may have: `binding/`, `controller/`, `data/`, `model/`, `ui/`, `widgets/` — use only what the feature needs
- Feature-specific repositories → `features/<name>/data/`
- Cross-cutting repositories → `core/repositories/`, registered in `main.dart` with `fenix: true`
- Feature-specific models → `features/<name>/model/`
- Shared/SDUI models → `core/model/`

### GetX Bindings (Fenix Convention)

- Every routed feature has a `Binding` class that registers controllers via `Get.lazyPut`
- Bindings are declared per-route in `core/routes/app_routes.dart`
- Global repositories in `main.dart` use `fenix: true` (survives controller disposal). Feature-local controllers do NOT use fenix.

### Result<T> + AppFailure Pattern

All API calls return `Result<T>` — a sealed class (`Ok<T>` / `Err<T>`) in `core/data/result.dart`. Typed failure hierarchy: `NetworkFailure`, `ServerFailure`, `ParseFailure`, `CancelledFailure`.

Controllers must pattern-match with Dart 3 switch:


```dart
switch (result) {
  case Ok(:final data): handleSuccess(data);
  case Err(:final failure): handleError(failure);
}
```

Never catch exceptions from API calls directly — `ApiClient` handles that internally.

### API Envelope


v2 endpoints return `{ meta, data }`. The **full envelope** is passed to model parsers (not just `data`). Use `ApiClient.safeGet`/`safePost` for v2 endpoints. Use `safeGetRaw` for legacy v1 endpoints without the envelope.

### Mock Fallback

Repositories may fall back to mock data on API failure. Place mock data under `features/<name>/data/mock/`.

### Server-Driven UI (SDUI)

Campus tab is server-driven. Section types defined in `core/model/sdui_section.dart`, rendered by widgets in `core/widgets/sdui/`. Unknown section types → `SizedBox.shrink()` for backward compatibility.

## Build, Run, Environment

```bash
flutter pub get                                            # Install dependencies
flutter run --dart-define-from-file=env/dev-ios.env        # iOS local dev
flutter run --dart-define-from-file=env/dev-android.env    # Android local dev
flutter run --dart-define-from-file=env/staging.env        # Staging
flutter build apk --dart-define-from-file=env/prod.env     # Android release
flutter build ios --dart-define-from-file=env/prod.env     # iOS release
flutter analyze                                            # Static analysis
flutter test                                               # Run all tests
```

**IMPORTANT:** Without `--dart-define-from-file`, the app defaults to **production API** (`https://api.skkuuniverse.com`). Always specify an env file.

### Two-Layer Environment System

- **Build-time** (`env/*.env`): Injects `BASE_URL` and `ENV` via `--dart-define-from-file` → accessed with `String.fromEnvironment`. Controls API server target.
- **Runtime** (`.env` in project root): SDK keys (Naver Map, AdMob) loaded by `flutter_dotenv`. Not committed to repo.

### VSCode Launch Configs

`.vscode/launch.json` has pre-configured profiles: **Dev (iOS)**, **Dev (Android)**, **Staging**, **Prod** — each with the correct `--dart-define-from-file` flag. Use these instead of manual CLI flags.

### OTA Updates

Shorebird is configured for code-push OTA updates (`shorebird.yaml`). Use `shorebird` CLI for patch releases.

## Testing

- **TDD workflow:** Write failing test → implement → make it pass → refactor
- Test files mirror source structure: `test/features/<name>/` and `test/core/`
- **Unit tests:** Repositories (mock ApiClient), controllers (mock repos), model parsing (JSON ↔ model round-trip)
- **Widget tests:** Screens with complex interaction logic
- Mock data: Reuse existing mock files under `features/<name>/data/mock/` or create test fixtures in `test/fixtures/`
- Run single test: `flutter test test/path/to/test.dart`
- Test naming: `<class_under_test>_test.dart`
- Use `Get.testMode = true` in test setUp for GetX controller tests

## Adding a New Feature

1. Create `lib/features/<feature_name>/` with subdirs as needed (`binding/`, `controller/`, `data/`, `model/`, `ui/`, `widgets/`)
2. Create `Binding` class → register controllers via `Get.lazyPut`
3. Add route constant to `Routes` in `core/routes/app_routes.dart`
4. Add `GetPage` with binding in the route list
5. For API calls: create repository returning `Result<T>` via `ApiClient.safeGet` with v2 envelope parser
6. If repository serves 2+ features → move to `core/repositories/`, register in `main.dart` with `fenix: true`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/spencer0124) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
