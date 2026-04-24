---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SpaceTraders is a Kotlin Multiplatform (KMP) SDK and Android client for the [SpaceTraders IO](https://spacetraders.io) REST API MMO. The full OpenAPI spec lives at `../OpenAPISpec/space_trader_open_api_spec.json` relative to this directory and is the authoritative reference for all API endpoints, request/response shapes, and auth schemes.

**Targets:** The SDK targets Android (minSdk 28) and iOS (arm64 + simulator). The Android app is a standalone Jetpack Compose application. An iOS app will consume the SDK's xcframework natively. Desktop/web are out of scope.

## Git Branching Strategy

This project follows Git Flow:

- **`main`** — production-ready releases only. Never commit directly.
- **`develop`** — integration branch. All feature/bugfix work merges here first.
- **`feature/<name>`** — branched from `develop`, merged back to `develop` via PR.
- **`bugfix/<name>`** — branched from `develop`, merged back to `develop` via PR.
- **`release/<version>`** — branched from `develop` when cutting a release, merged into both `main` and `develop`.
- **`hotfix/<name>`** — branched from `main` for critical production fixes, merged into both `main` and `develop`.

```bash
# Start a new feature
git checkout develop && git pull
git checkout -b feature/my-feature

# Finish and push for PR
git push -u origin feature/my-feature
# Open PR targeting develop (GitHub default)
```

`develop` is the default branch on GitHub — all PRs target it automatically.

## Build Commands

All commands run from the `KMP/` directory. On Windows use `gradlew.bat` instead of `./gradlew`.

```bash
# Build Android debug APK
./gradlew :app:assembleDebug

# Compile SDK (catches type errors before running tests)
./gradlew :spacetradersiosdk:compileAndroidHostTestSources

# Run all SDK tests (androidHostTest + commonTest compiled for JVM)
./gradlew :spacetradersiosdk:allTests

# Run app unit tests (ViewModels, etc.)
./gradlew :app:testDebugUnitTest

# Run a single SDK test class
./gradlew :spacetradersiosdk:testAndroidHostTest --tests "com.brokenhuskysledteam.spacetradersio.sdk.MyTest"

# Run a single app test class
./gradlew :app:testDebugUnitTest --tests "com.brokenhuskysledteam.spacetradersio.ui.ships.ShipDetailViewModelTest"
```

iOS framework is built via `./gradlew :spacetradersiosdk:assembleSpacetradersiosdkKitReleaseXCFramework`. There is no iOS app in this repo yet — the SDK produces an xcframework (`spacetradersiosdkKit`) for consumption by a native Swift/Xcode project.

## Architecture

Two Gradle modules: `:spacetradersiosdk` (KMP shared library) and `:app` (Android entry point).

`:spacetradersiosdk` has three source sets:

- **`commonMain`** — all shared logic: API layer, domain models, use cases, data repositories. No UI code.
- **`androidMain`** — Platform actuals (`Platform.android.kt`) and Android Ktor engine (OkHttp).
- **`iosMain`** — Platform actuals (`Platform.ios.kt`) and iOS Ktor engine (Darwin).

`:app` is a standalone Android application using Jetpack Compose (not Compose Multiplatform). It owns all UI, ViewModels, navigation, and DI. It depends on `:spacetradersiosdk` for business logic.

The `expect/actual` mechanism in `Platform.kt` is the current example of platform-specific behaviour.

### App layer structure inside `app/src/main`

```
di/          <- Hilt module bridging SDK types into Android DI graph
navigation/  <- Navigation Compose routes (@Serializable objects) and NavHost
ui/auth/     <- Auth screen: register + token import (UDF: StateFlow + sealed events)
ui/dashboard/ <- Dashboard screen: agent info, logout (UDF: StateFlow + sealed events)
ui/components/ <- Reusable themed composables: TerminalCard, TerminalButton, TerminalTextField, TerminalProgressBar, ScanlineOverlay
ui/theme/    <- Retro terminal theme (dark-only, green-on-black, monospace, sharp corners)
ui/ships/    <- Ship list + detail screens (UDF: StateFlow + sealed events)
```

ViewModels use `Channel<NavigationTarget>` (not SharedFlow) for one-shot navigation to avoid re-delivery on config change.

The app uses a dark-only retro terminal aesthetic. Design inspiration images live at `../InsipirationImages/` (note: folder name has a typo). New UI should follow this visual language: green-on-black, bordered panels, monospace text, uppercase headers.

### Layer structure inside `spacetradersiosdk/src/commonMain`

```
api/        <- Ktor HttpClient, endpoint functions, request/response DTOs
api/mapper/ <- DTO-to-domain-model extension functions
domain/     <- Business models, use cases, repository interfaces
data/       <- Repository implementations
```

### Error handling


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MarshallLadd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
