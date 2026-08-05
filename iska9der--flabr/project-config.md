---
trigger: always_on
description: This file provides guidance to Agents when working with this repository.
---

# AGENTS.md

This file provides guidance to Agents when working with this repository.

## Project Overview

Flabr is an unofficial mobile client for habr.com built with Flutter. It supports Android, iOS, and Web platforms with features including dark/light themes, authorization, feed customization, and AI-powered article summaries using YandexGPT.

**Architecture**: Clean Architecture with BLoC pattern
**Key Technologies**: Flutter 3.41.0, Dart 3.11, auto_route, injectable/get_it, freezed

## Quick Reference

- **Architecture Details**: [docs/architecture/overview.md](docs/architecture/overview.md)
- **Development Commands**: [docs/development/commands.md](docs/development/commands.md)
- **Code Style**: [docs/development/code-style.md](docs/development/code-style.md)
- **Common Tasks**: [docs/development/common-tasks.md](docs/development/common-tasks.md)

## Mandatory Workflow

For non-trivial code changes, do not rely only on this AGENTS.md summary. Before editing, inspect the relevant documentation and nearby implementation examples.

### Required Docs Lookup

- Read [docs/architecture/overview.md](docs/architecture/overview.md) before changing layer boundaries, repositories, services, DI, BLoC/Cubit, routing, app initialization, or shared infrastructure.
- Read [docs/architecture/presentation-layer.md](docs/architecture/presentation-layer.md) before changing screens, widgets, UI state handling, themes, or responsive behavior.
- Read [docs/architecture/core-components.md](docs/architecture/core-components.md) before changing router, logger, shortcuts, storage, HTTP, or other shared core components.
- Read [docs/development/code-style.md](docs/development/code-style.md) before making Dart style, naming, decomposition, comment, import, or formatting decisions.
- Read [docs/development/common-tasks.md](docs/development/common-tasks.md) before adding features, screens, routes, repositories, models, DI entries, or generated-code-backed types.
- Read [docs/development/commands.md](docs/development/commands.md) before running project commands.
- Read the relevant file in [docs/flows/](docs/flows/) before changing authentication, app initialization, or adjacent flows.
- Read [docs/packages/README.md](docs/packages/README.md) before changing workspace packages or package boundaries.

### Before Coding Checklist

For any non-trivial change:
1. Read the target file.
2. Read 2-3 nearby files that solve similar problems.
3. Read the relevant `docs/` page listed above.
4. Identify whether generated files, DI, routes, models, or workspace packages are affected.
5. State the intended edit briefly before patching.

## Quality Bar

A change is not done until it is consistent with local project patterns, not just generic Flutter/BLoC practices.

- Prefer existing local abstractions, naming, state shape, and widget composition over introducing new patterns.
- Keep changes in the smallest layer that owns the behavior; do not move logic across architecture boundaries without a clear reason.
- Do not introduce a new abstraction unless it removes meaningful duplication, reduces real complexity, or matches an established local pattern.
- UI changes should handle loading, empty, error, and success states where applicable, using the same conventions as neighboring screens.
- Data and domain changes should preserve repository/service boundaries and existing error-handling style.
- State management changes should use existing BLoC/Cubit conventions and the shared `LoadingStatus` enum.
- Generated files must be updated only through the appropriate generator command.
- For non-trivial Dart changes, run formatter and analyzer through FVM before finishing, unless the user explicitly asks not to.

## Critical Instructions

**IMPORTANT**: This project uses FVM (Flutter Version Manager). All `flutter` and `dart` commands must be executed through `.fvm/flutter_sdk/bin/` path. Current Flutter version is defined in `.fvmrc` and `pubspec.yaml`.

### Code Generation

After modifying models, DI annotations, or routes, always run:
```bash
.fvm/flutter_sdk/bin/flutter pub run build_runner build --delete-conflicting-outputs
```

Do not edit generated files manually:
- `*.g.dart`
- `*.freezed.dart`
- `*.gr.dart`
- `*.config.dart`

### BLoC/Cubit Usage

- **Never use `await`** when calling Cubit/Bloc methods from UI
- Call methods without await (fire-and-forget)
- If a returned `Future` must be intentionally ignored outside UI callbacks, use `unawaited(...)` instead of leaving it implicit
- Use `BlocListener` or `BlocBuilder` to react to state changes
- Always use shared `LoadingStatus` enum from `lib/data/model/loading_status_enum.dart`

### Documentation Comments

- Use triple-slash `///` for documentation comments (Dart convention)
- Write comments in Russian, keep technical terms in their original language
- Write comments for future readers, not for the current task context: explain non-obvious behavior, invariants, lifecycle constraints, or why a piece of code must stay that way.
- Do not add comments that merely restate the code, describe a recent refactor, or compare the current implementation with a removed/alternative approach.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iska9der/flabr](https://github.com/iska9der/flabr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
