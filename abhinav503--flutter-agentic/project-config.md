---
trigger: always_on
description: Project-wide coding conventions and rules for FlutterAgentic. Always include for all code generation tasks.
---


## Documentation Index

Read before writing or modifying any code:
- `docs/reference/architecture.md` — core folder map, layer patterns, naming, DI, error flow, design system, testing
- `docs/explanation/end-goal.md` — project vision and guiding principles

Read on demand:
- `docs/how-to/contributing.md` — contributor workflow and git hooks
- `docs/how-to/add-feature-template.md` — full folder tree, empty class skeletons, DI wiring, and forbidden-pattern checklist for scaffolding a new feature
- `docs/how-to/add-usecase.md` — create a use case class and register it in `injection_container.dart`
- `docs/how-to/design-screen-state.md` — business-logic naming for events and states, retry context rules, screen rendering pattern; use the jokes feature as the reference
- `docs/how-to/review-code.md` — when asked to review, audit, or check generated code; run through the full checklist and report ✅/❌ per section
- `docs/how-to/change-app-id.md` — when asked to change the application ID or bundle identifier; covers Android (`build.gradle.kts` + `MainActivity.kt` package path) and iOS (`project.pbxproj`), with Xcode manual steps and provisioning notes
- `docs/how-to/rename-app.md` — when asked to rename the app; covers display name, package name, and all files that reference the old name
- `docs/how-to/connect-firebase.md` — when connecting an app to Firebase; covers checking/installing the Firebase + FlutterFire CLIs, running `flutterfire configure`, per-app `firebase_core`, `main.dart` init, Android Gradle plugin, iOS deployment target (15.0+), and the Xcode `GoogleService-Info.plist` registration check
- `docs/tutorials/solid-principles.md` — how SOLID principles are applied across all layers; useful when designing new classes or reviewing layer boundaries
- `docs/tutorials/design-patterns-and-concepts.md` — design patterns used in this codebase (Singleton, Repository, DTO, Either, Sealed Classes, Strategy, and more)
- `docs/explanation/ai-agents.md` — per-agent install and usage
- **Release workflow** — see `release.mdc` in this rules folder

---

## Monorepo Layout

Dart pub-workspace monorepo: one shared `core` package consumed by multiple Flutter apps.

```
packages/core/   shared toolbelt → import 'package:core/core/…'   (no app-specific code)
apps/jokes/      demo app          apps/doc_scanner/  request/response app
apps/ai_chat/    streaming app
```

One `flutter pub get` at the repo root resolves all packages; editing `core` is live in any running app. Each app owns its `main.dart`, `app.dart`, `di/injection_container.dart`, `constants/` (`ValueConst`/`ApiConstants`), and `feature/home/`; `core` holds only `CoreConst`. Run `make` targets from the repo root; run an app from its folder (`apps/<app>`).

---

## Build & Run

After cloning, run `make setup` once to install git hooks and fetch packages.

```bash
make setup            # first-time setup: git hooks + root flutter pub get
make run-jokes        # run the jokes app (cd apps/jokes && flutter run)
make run-doc-scanner  # run the doc_scanner app
make run-ai-chat      # run the ai_chat app
make web-jokes        # run jokes on Chrome
make test             # flutter test in each app
make analyze          # flutter analyze — whole workspace
make gen              # build_runner in core + each app
make clean            # flutter clean per package, then root pub get
```

The pre-commit hook formats staged Dart files and runs `flutter analyze` at the root — commits are blocked if analysis fails.

---

## Forbidden Patterns

- Hardcoded colours, strings, spacing, or radii in widget files
- Business logic in `build()` or widget classes
- `import 'package:dio/...'` from `domain/`
- `if (state is XState)` — always use exhaustive `switch`
- Giving an enum methods/fields in its own body (enhanced enum) when an `extension on` it would do — keep enums as bare case lists; put helpers, `switch` mappings, and string ↔ enum conversion in an extension beside the enum. Likewise, put repeated `String`/`num`/`DateTime` logic in an `extension on` that type, not a `*Utils` helper class or inline. Don't add a conversion extension for an enum that never crosses a string boundary; parse wire strings in the `*Model` (data layer), not the UI
- Comments that restate what the code or name already says — with business-logic naming most doc comments are redundant. Write **why** (non-obvious decisions, gotchas, constraints), not **what**. Don't repeat the same note in two places, and reserve longer comments for genuinely complex logic. Examples:
  - ❌ `/// Stops the app.` above `Future<…> stopApp(String name)` — the name says it
  - ❌ `/// Param is the app name.` above `RunAppUseCase` — the signature says it
  - ✅ `// Not const — owns the live WebSocket the other methods act on`
  - ✅ `// Returning a state equal to the current one is a no-op, so per-chunk updates don't rebuild`
- `context.read<T>()` after an `await` without a `mounted` check
- More than one feature's logic in a single BLoC
- Exposing `*Model` classes outside the `data/` layer
- Calling `AppBottomSheet.show()` directly in a screen — use `showAppBottomSheet()` from `BaseScreenState`
- Extending `StatelessWidget`/`StatefulWidget` directly for full pages or screens

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abhinav503/flutter-agentic](https://github.com/abhinav503/flutter-agentic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
