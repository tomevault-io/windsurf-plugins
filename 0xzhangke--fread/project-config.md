---
trigger: always_on
description: This file is the operating manual for AI coding agents working in the Fread repository.
---

# AGENTS.md

This file is the operating manual for AI coding agents working in the Fread repository.
Keep instructions concrete, executable, and specific to this codebase.

## Project Overview

Fread is a Kotlin Multiplatform decentralized microblogging client for Mastodon, Bluesky, and RSS. The main implementation is Kotlin/Compose Multiplatform with Android and iOS targets.

Important modules:

- `app`: Android application wrapper, signing, packaging, launcher resources, and APK renaming.
- `iosApp`: Swift/iOS host project for the shared Compose app.
- `app-hosting`: shared app shell, root navigation, Koin startup, platform app entry points, and `FreadKit` framework output for iOS.
- `framework`: shared infrastructure utilities, Compose helpers, HTTP/client primitives, permissions, parsing, persistence helpers, and platform `expect`/`actual` abstractions.
- `bizframework/status-provider`: status-provider contracts, account abstractions, rich text parsing, status models, publishing/search interfaces, and protocol-facing types.
- `commonbiz/common`, `commonbiz/analytics`, `commonbiz/status-ui`, `commonbiz/sharedscreen`: shared business logic, analytics abstractions, reusable status UI, and shared screens.
- `feature/*`: user-facing feature modules such as feeds, explore, profile, and notifications.
- `plugins/activitypub-app`, `plugins/bluesky`, `plugins/rss`: protocol implementations integrated through status-provider contracts.
- `plugins/Fread-Firebase`: optional Firebase/push/review integration. It is included only when the module exists and `-PdisableFirebase=true` is not passed.
- `localization`: Compose Multiplatform resources and localization helpers.
- `thirds/*`: vendored rich text libraries. Treat these as third-party code and avoid edits unless the task is specifically about them.
- `build-logic`: included Gradle build containing Fread convention plugins. Prefer using or extending existing convention plugins over duplicating Gradle setup in modules.

## Architecture Rules

- Keep protocol-specific behavior in its plugin module. Do not leak Mastodon/ActivityPub, Bluesky, RSS, or Firebase implementation details into generic `framework` or `commonbiz` modules unless the abstraction is intentionally shared.
- Put cross-provider status contracts and models in `bizframework/status-provider`; put reusable UI rendering in `commonbiz/status-ui`; put feature workflows in `feature/*`.
- Use `app-hosting` for composition of the app shell, root navigation, and top-level dependency graph. Keep platform launch wrappers thin in `app` and `iosApp`.
- Prefer `commonMain` for shared behavior. Add `expect` declarations in `commonMain` and `actual` implementations in `androidMain`/`iosMain` only for real platform differences.
- Keep build configuration centralized in `build-logic` and `gradle/libs.versions.toml`. Avoid one-off Gradle logic unless it is truly module-specific.
- Do not add broad dependencies to foundational modules casually. First check whether the dependency belongs in a feature or plugin module.
- Preserve the optional Firebase design: code must still build with `-PdisableFirebase=true` when the changed area does not require Firebase.

## Kotlin And Compose Style

- Use Kotlin idioms already present in the repo: immutable `data class` UI state, sealed interfaces/classes for state variants, extension functions for small reusable operations, and explicit constructor injection.
- Compose functions should be small, state-hoisted where practical, and named with PascalCase when they emit UI.
- Use `remember`, `rememberSaveable`, `derivedStateOf`, `LaunchedEffect`, and `DisposableEffect` deliberately. Avoid hidden mutable state that survives longer than the composable it belongs to.
- Keep ViewModels free of direct platform UI dependencies. Use injected collaborators, `viewModelScope`, and existing helpers such as `launchInViewModel` where appropriate.
- Follow existing formatting: 4-space indentation, trailing commas in multiline calls/classes where local code uses them, and explicit named parameters for readability in Compose calls.
- Prefer existing framework helpers before adding new utilities.
- For user-visible text, use localization resources from `localization/src/commonMain/composeResources` and `LocalizedString` patterns instead of hardcoded strings.
- For UI changes, check Android and iOS source sets for platform-specific implementations before assuming common code is enough.

## Dependency Injection And Navigation

- Use Koin modules following existing `val <name>Module = module { ... }` patterns.
- For platform-specific bindings, keep the common declaration as `expect fun Module.createPlatformModule()` and implement Android/iOS `actual` functions in the matching source set.
- Register new feature/plugin dependencies in the owning module first, then wire them through `app-hosting` only when they must participate in app startup.
- Respect existing Navigation 3 and KRouter patterns. If adding generated KRouter pieces, ensure KSP is configured for all needed KMP targets with `kspAll(...)` and `configureCommonMainKsp()` where applicable.

## Gradle And Dependencies

- Use the version catalog in `gradle/libs.versions.toml` for external dependencies.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0xZhangKe/Fread](https://github.com/0xZhangKe/Fread) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
