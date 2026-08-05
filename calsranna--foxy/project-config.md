---
trigger: always_on
description: This file applies to the whole repository. Foxy is a Flutter desktop application for visually editing AzerothCore 3.3.5a data. It connects to a MySQL world database, edits normal AzerothCore tables, and imports/exports client DBC data through tables in a separate `foxy` schema. The UI and most user-facing errors are Chinese.
---

# AGENTS.md

## Purpose and scope

This file applies to the whole repository. Foxy is a Flutter desktop application for visually editing AzerothCore 3.3.5a data. It connects to a MySQL world database, edits normal AzerothCore tables, and imports/exports client DBC data through tables in a separate `foxy` schema. The UI and most user-facing errors are Chinese.

`README.md` provides the user/developer overview and setup instructions. There is no CI configuration; use the source, contract tests, `pubspec.yaml`, `README.md`, and this file together as the repository documentation.

## Cold start

Requirements:

- Flutter stable with Dart compatible with `sdk: ^3.9.2` (`.metadata` records the Flutter project revision).
- A desktop toolchain for the target platform. Tracked runners exist for Linux, macOS, and Windows.
- MySQL only for running the application and the opt-in integration tests; the normal test suite is primarily hermetic.

Run commands from the repository root. Several tests inspect files through relative paths such as `lib/entity`, so invoking them from another working directory can give misleading failures.

```bash
flutter pub get
flutter analyze
flutter test
flutter run -d macos       # or windows/linux when available
```

The application reads and writes `config.yaml` relative to `Directory.current`, so launch it from the project root during development. That file contains connection settings, including the password, and is intentionally gitignored. Never commit it or credentials.

Useful focused commands:

```bash
flutter test test/<name>_test.dart
flutter test test/<name>_test.dart --plain-name '<exact test name>'
dart format <changed .dart files>
dart run build_runner build --delete-conflicting-outputs
flutter build macos        # or windows/linux
```

Do not globally format the repository as incidental cleanup. Format only changed Dart files and keep diffs surgical.

## Repository map

- `lib/main.dart` — Flutter entry point; initializes the desktop window, DI, and the app.
- `lib/di.dart` — central `get_it` registration for repositories, utilities, and view models.
- `lib/database/` — connection singleton and append-only application migrations.
- `lib/infrastructure/`
  - `config/` — root-relative YAML configuration.
  - `database/` — MySQL-specific error helpers.
  - `dbc/` — DBC definitions, import worker, sync orchestration, export encoding, locale codecs, and export registry.
  - `logging/`, `preferences/`, `util/`, `window/` — shared infrastructure.
- `lib/entity/` — immutable full entities, brief/list entities, and typed composite/special row keys.
- `lib/repository/` — Laconic query and persistence layer plus Repository-owned Filter query models. Most repositories map one physical table or a tightly related table family.
- `lib/use_case/` — concrete user-intent operations for cross-repository orchestration, transactions, migrations, and cancellable workflows.
- `lib/page/<module>/` — MVVM feature folders containing pages, views, view models, and occasional module validation.
- `lib/widget/` — shared shadcn-based widgets, form controllers, pickers, locale editors, tables, and dialogs.
- `lib/constant/` — AzerothCore/DBC enums, flags, schemas, and UI option definitions.
- `lib/router/` — AutoRoute configuration, navigation facade/menu model, and generated routes.
- `lib/event/` — small synchronous application event bus.
- `test/` — unit/widget tests plus extensive source-level architecture and database-editing contract tests.
- `asset/image/` — Flutter-bundled app images.
- `asset/icon/` — thousands of game icons deliberately kept out of the Flutter asset bundle.
- `linux/`, `macos/`, `windows/` — desktop runners and packaging configuration.

`build/`, `.dart_tool/`, Flutter plugin metadata, IDE state, logs, and `config.yaml` are generated/local artifacts; do not edit or commit them.

## Architecture and data flow

The normal feature flow is:

1. A `@RoutePage()` page obtains a factory-created view model from `GetIt`, calls `initSignals()` in `initState`, and disposes it with the page.
2. A view renders shadcn widgets and delegates behavior to the view model.
3. The view model owns Signals state and registered field controllers, validates a collected candidate, and calls a repository.
4. The repository maps physical SQL rows to entities through Laconic.
5. `RouterFacade` keeps AutoRoute navigation and the signal-backed breadcrumb/menu path in sync.

State management uses `signals`/`signals_flutter`; reactive widget regions use `Watch`. Dependencies are obtained through `GetIt.instance`. Register infrastructure, repositories, use cases, global state ViewModels, and interaction ViewModels in that order in `lib/di.dart`.

Every ViewModel belongs to exactly one closed category and declares it in both
the class and file suffix: `ListViewModel`, `DetailViewModel`,
`CollectionEditorViewModel`, `SingleEditorViewModel`, `ReadViewModel`,
`WorkflowViewModel`, or `StateViewModel`. Do not add an unclassified

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CalsRanna/foxy](https://github.com/CalsRanna/foxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
