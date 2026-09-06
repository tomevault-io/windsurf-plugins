---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

爱看书 — open-source Flutter novel reader. Package name: `book`. Application ID: `com.opensource.ikanshu`. License: Apache-2.0.

Tech stack: Dart 3 / Flutter 3.44+, **Riverpod** (`flutter_riverpod` + `ChangeNotifierProvider`), event_bus, Dio 5, Fluro, sqflite, protobuf, shared_preferences (via local `SpUtil` facade).

**SDK constraint:** `sdk: ^3.12.0` — sound null safety. Target Flutter 3.44 stable (Dart 3.12).

## Commands

```bash
# Install dependencies
flutter pub get

# Analyze (flutter_lints via analysis_options.yaml)
flutter analyze

# Run on a connected device/emulator
flutter run

# Run tests
flutter test
flutter test test/widget_test.dart   # single test file

# Debug APK (requires a valid Android SDK / ANDROID_HOME)
flutter build apk --debug

# Release APK (CI-style)
flutter build apk

# Release APK matching build.bat (obfuscated, arm64 split, SKSL warm-up)
flutter build apk --obfuscate --split-debug-info=HLQ_Struggle --target-platform android-arm64 --split-per-abi --build-name=4.2.3 --build-number=3 --bundle-sksl-path flutter_01.sksl.json

# Regenerate json_serializable code (optional; *.g.dart are also hand-maintained)
dart run build_runner build --delete-conflicting-outputs
```

`build.bat` / `build_arm64.bat` are the project’s release build scripts (Windows).

GitHub Actions:
- `.github/workflows/ci.yml` — `flutter analyze` + `flutter test` on push/PR
- `.github/workflows/build.yml` — release APK on `master`/`main`, tags `v*`, `workflow_dispatch`, and `repository_dispatch` type `starred`; optional signing via `ANDROID_KEYSTORE_*` secrets; uploads artifacts and creates a GitHub Release for tags

There is no separate lint script beyond `flutter analyze`. `file_names` is enabled (all lib paths are snake_case). Residual SpUtil/API-shaped identifiers still suppress `non_constant_identifier_names` / `constant_identifier_names`.

`pubspec.yaml` uses `dependency_overrides.platform: ^3.1.6` because transitive `sqflite_platform_interface` otherwise pulls `platform 3.0.0`, which references removed `io.Platform.packageRoot` on Dart 3.12.

## Architecture

### Boot sequence

1. `main()` → `AppInit.init()` then `runApp(const ProviderScope(child: MyApp()))`.
2. `AppInit` (`lib/app_init.dart`) requests media/storage permission (mobile), initializes Firebase Analytics + Crashlytics (`FirebaseBootstrap` in `lib/service/firebase_bootstrap.dart`), initializes local `SpUtil` (`lib/common/local_store.dart`), registers `TelAndSmsService` on global `GetIt` (`locator` in `lib/main.dart`), configures Fluro (`Routes`), loads package version.
3. `MyApp` is a `ConsumerWidget` watching `colorModelProvider` for theming; home is `MainShell` (书架 / 发现 / 我); routes via `Routes.router.generator`; toasts via BotToast; screen views via `FirebaseAnalyticsObserver`.

### State management

- **Riverpod** via `lib/store/providers.dart` (providers only; no `Store` facade class):
  - Root: `ProviderScope` in `main.dart`
  - Providers: `searchModelProvider`, `exploreModelProvider`, `colorModelProvider`, `shelfModelProvider`, `readModelProvider`, `sourceModelProvider` (`ChangeNotifierProvider`)
  - Models remain `ChangeNotifier` subclasses under snake_case files (`search_model.dart`, `explore_model.dart`, `color_model.dart`, `shelf_model.dart`, `read_model.dart`, `source_model.dart`)
  - UI uses `ConsumerWidget` / `ConsumerStatefulWidget` with `ref.watch` / `ref.read` directly.
- **event_bus** (`lib/event/event.dart`) for cross-widget signals (reading progress, shelf sync, page controller, download progress, etc.). Global: `eventBus`.
- **GetIt** for a few services (`locator`), not for the main UI models.

### Layers

| Path | Role |
|------|------|
| `lib/view/book/` | Core UI: shelf, search, detail, reader (`read_book.dart`), chapters, sort shelf |
| `lib/view/page_turn/` | Canvas page-turn: `novel_page_painter.dart`, `reader_page_manager.dart` |
| `lib/view/person/` | Account: login, register, me, skin, cache |
| `lib/view/system/` | Reader chrome: font, menu, battery, log viewer |
| `lib/model/` | `ChangeNotifier` business logic (snake_case: `read_model.dart`, `shelf_model.dart`, …) |
| `lib/entity/` | DTOs: `json_annotation` + checked-in `*.g.dart` (camelCase fields; no legacy JSON key compat) |
| `lib/common/` | Shared infra: API URLs (`common.dart`), Dio (`http.dart`), text layout (`text_composition.dart`), interceptors, `screen.dart`, **`local_store.dart` (SpUtil/DateUtil/NumUtil)** |
| `lib/data/` | Local persistence: `ReaderDatabase` (`reader.db`), `BookRepository`, `ChapterRepository`, `SourceRepository` |
| `lib/route/` | Fluro route table (`routes.dart`) and handlers (`route_handler.dart`) |
| `lib/animation/` | Custom page-turn animations used by the reader |
| `lib/widgets/` | Reusable UI pieces |
| `lib/service/` | Cache manager, tel/SMS helper |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leetomlee123/book](https://github.com/leetomlee123/book) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
