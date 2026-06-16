---
trigger: always_on
description: This file is the canonical repository reference for coding agents working in this project.
---

# AGENTS.md

This file is the canonical repository reference for coding agents working in this project.

## Project Summary

Practical Filament is sample code for a talk on Google's Filament 3D rendering framework. The codebase is a Kotlin Multiplatform mobile app targeting Android and iOS with shared UI built in Compose Multiplatform.

Current repo snapshot:
- Workspace git state is not stable; always check `git status` and `git branch --show-current` before assuming branch or cleanliness
- Gradle wrapper verified with `./gradlew help`
- Gradle 9.3.1
- Kotlin 2.3.20
- Compose Multiplatform 1.10.3
- Android Gradle Plugin 9.1.0
- Filament 1.71.4
- The current demo is a metadata-driven material viewer that introspects `.filamat` parameters at runtime and builds controls dynamically

## Repository Structure

- `composeApp/` shared Kotlin Multiplatform module
- `androidApp/` Android application host project
- `iosApp/` native iOS host project and Xcode project
- `tools/filament/<version>/ios/` vendored iOS Filament SDK headers and static libraries used by the Xcode project (version from `gradle/libs.versions.toml`)
- `gradle/libs.versions.toml` centralized dependency and plugin versions

## Architecture

The shared application logic lives in `:composeApp` and is split across these source sets:

- `commonMain` shared Compose UI, Filament abstractions, theme, screens, typed material parameter models, and helper texture generators
- `androidMain` Android-specific implementations such as `MainActivity`, platform bindings, and the Filament Android engine/view integration
- `iosMain` iOS-specific implementations such as the Compose entry point, `CAMetalLayer` host, and Kotlin-side bridge protocol used by Swift/ObjC++
- `commonTest` shared tests

Platform-specific behavior uses Kotlin `expect`/`actual` declarations where needed.

The Filament integration is organized in three layers:
- Shared KMP API in `composeApp/src/commonMain/kotlin/dev/nstv/practicalfilament/filament/` via `FilamentEngine`, `FilamentView`, `MaterialParameter`, and `MaterialParameterType`
- Android implementation in `composeApp/src/androidMain/kotlin/dev/nstv/practicalfilament/filament/AndroidFilamentEngine.kt` using Filament's Android/JVM API, `SurfaceView`, `UiHelper`, and a `Choreographer` render loop
- iOS implementation split between Kotlin (`IosFilamentEngine`, `FilamentView.ios.kt`), Swift (`FilamentBridgeAdapter.swift`), and ObjC++ (`iosApp/iosApp/FilamentBridge.h` / `.mm`) over a Metal-backed Filament engine

Runtime material editing is metadata-driven:
- `FilamentEngine.getMaterialParameters(materialHandle)` returns the declared Filament parameter definitions from the compiled `.filamat`
- `composeApp/src/commonMain/kotlin/dev/nstv/practicalfilament/components/ParameterInputField.kt` renders controls based on `MaterialParameterType`
- `composeApp/src/commonMain/kotlin/dev/nstv/practicalfilament/screen/MaterialViewerScreen.kt` loads a material, creates defaults from the metadata, and applies edits back through `setMaterialParameter` or `setTextureParameter`

## Entry Points

- Android: `composeApp/src/androidMain/kotlin/dev/nstv/practicalfilament/MainActivity.kt`
- Shared app root: `composeApp/src/commonMain/kotlin/dev/nstv/practicalfilament/App.kt`
- Shared main screen: `composeApp/src/commonMain/kotlin/dev/nstv/practicalfilament/screen/MainScreen.kt`
- Material viewer demo: `composeApp/src/commonMain/kotlin/dev/nstv/practicalfilament/screen/MaterialViewerScreen.kt`
- iOS Compose host: `composeApp/src/iosMain/kotlin/dev/nstv/practicalfilament/MainViewController.kt`
- iOS Swift host: `iosApp/iosApp/ContentView.swift`
- iOS Swift adapter wiring: `iosApp/iosApp/FilamentBridgeAdapter.swift`

## Build And Test Commands

```bash
# Project help / wrapper verification
./gradlew help

# Build everything
./gradlew build

# Android
./gradlew assembleDebug
./gradlew assembleRelease

# iOS binaries
./gradlew iosArm64Binaries
./gradlew iosSimulatorArm64Binaries

# Tests
./gradlew test
./gradlew testDebug
./gradlew iosSimulatorArm64Test
```

For iOS app development, open `iosApp/iosApp.xcodeproj` in Xcode. The current project is linked against the vendored Filament SDK in `tools/filament/<version>/ios` (version defined in `Config.xcconfig` as `FILAMENT_VERSION`).

## Notable Implementation Areas

- `composeApp/src/commonMain/kotlin/dev/nstv/practicalfilament/screen/` main app screens, including the material viewer and material-selection workflow
- `composeApp/src/commonMain/kotlin/dev/nstv/practicalfilament/components/ParameterInputField.kt` dynamic parameter controls for scalars, vectors, matrices, booleans, integer types, arrays, and sampler inputs
- `composeApp/src/commonMain/kotlin/dev/nstv/practicalfilament/filament/FilamentEngine.kt` shared rendering/material/texture API
- `composeApp/src/commonMain/kotlin/dev/nstv/practicalfilament/filament/MaterialParameter.kt` shared material parameter definitions, typed parameter model, default values, and generated built-in textures
- `composeApp/src/androidMain/kotlin/dev/nstv/practicalfilament/filament/` Android Filament engine, material introspection, texture upload, and Android view integration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nicole-terc/PracticalFilament](https://github.com/nicole-terc/PracticalFilament) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
