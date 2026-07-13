---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Asuka Player is a clean-room Android local video player rewrite built with Jetpack Compose and Media3/ExoPlayer. The project is organized as a multi-module Gradle project with strict separation of concerns across 9 modules.

## Build & Development Commands

```bash
# Compile Kotlin for debug
./gradlew :app:compileDebugKotlin

# Run all JVM unit tests (fast, no device needed)
./gradlew test

# Compile AndroidTest sources without a device
./gradlew :player-ui:compileDebugAndroidTestKotlin

# Run a single test class
./gradlew :player-contract:testDebugUnitTest --tests "com.asuka.player.contract.PlaybackSessionPlannerTest"

# Lint
./gradlew lintDebug

# Run UI/instrumented tests (requires connected device or emulator)
./gradlew :player-ui:connectedAndroidTest

# Install debug APK on connected device
./gradlew :app:installDebug

# Full local verification (no device)
./gradlew test :player-ui:compileDebugAndroidTestKotlin lintDebug
```

## Module Architecture

```
app/                  → Library browsing, settings, file picker; launches PlaybackActivity
player-contract/      → Stable playback contracts/models and planning chain (package: contract)
player-platform/      → Android / Media3 binding API, intent/seek fallback, async writer
player-render-api/    → Playback surface / renderer neutral contracts
player-renderer/      → PlaybackActivity, session assembly, PiP, Media3 surface adapter
player-runtime/       → Settings repositories, runtime graph, launch coordination (package: runtime)
player-ui/            → Compose playback UI, gesture orchestration, UI state translation
player-engine/        → PlaybackService, Media3 controller/connector (package: engine)
player-domain/        → Pure JVM algorithms (no Android deps) — gesture math & state machines
player-data/          → DataStore/Room persistence, media library index, legacy migration
```

Dependency direction: `app` → `player-runtime` / `player-platform` / `player-renderer` / `player-data` / `player-engine`; `player-renderer` → `player-render-api` / `player-ui` / `player-platform` / `player-contract`; `player-ui` → `player-render-api` / `player-contract` / `player-domain`; `player-runtime` → `player-contract` / `player-platform` / `player-data`; `player-engine` → `player-contract` / `player-platform`

`player-domain` is a pure JVM library. `player-contract`, `player-data`, `player-engine`, `player-platform`, `player-renderer`, `app`, and `player-ui` all have JVM/Robolectric test coverage.

## Key Architecture Decisions

**Gesture system is split across two layers:**
- `player-domain/GestureAlgorithms.kt` — pure math (seek delta, zoom, brightness/volume mapping). No side effects, fully unit tested.
- `player-ui/controller/GestureCoordinator.kt` — coordinates gesture types, updates UI state and calls `PlaybackController`.
- `player-ui/controller/GestureStateMachine.kt` (in domain) — state machine that enforces gesture exclusivity.

**Application wiring:**
1. `AsuraPlayerApp` builds `AsukaAppGraph`, injecting engine bindings (`Media3PlaybackControllerConnectorFactory`, `PlaybackService` component, notification icon).
2. `AppComposition` wires the graph into narrow dependency interfaces (`PlaybackActivityDependencies`, `PlaybackServiceDependencies`) using inline anonymous objects.
3. Framework components resolve dependencies via `PlaybackDependenciesProvider.from(application)` / `MainActivityDependenciesProvider.from(application)` — centralized lookup with diagnostic error messages, validated by architecture boundary checks at build time.

**Playback state split:**
- `PlaybackHostState` holds slow-changing state (controller, connection, settings, PiP, degradation) — emits only on lifecycle events.
- `PlayerUiState` (progress, title, playing, buffering, error) flows at 20Hz through a separate `StateFlow` via `PlaybackViewModel.uiState`.
- `PlayerScreen` accepts `uiStateFlow: StateFlow<PlayerUiState>` and collects it internally, so the Activity composable does not recompose at 20Hz.

**Playback launch and control flow:**
1. `MainActivity` uses `PlaybackLaunchCoordinator` to resolve the playback URI and forward/remap explicit `ClipData` queues.
2. `PlaybackActivity` connects to `PlaybackService` (a `MediaSessionService`) via `MediaController`.
3. `PlaybackViewModel` holds `PlaybackSessionHost`, `PlaybackActivityBehavior`, and playback host state; survives configuration changes via `AndroidViewModel`.
4. `PlaybackLaunchOrchestrator` handles runtime policy, current launch intent, seek fallback, and artwork restore.
5. `PlaybackSessionCoordinator` asks `PlaybackSessionPlanner` for a `PlaybackSessionPlan` and applies it to the controller.
6. `Media3PlaybackController` wraps `MediaController` and implements the `PlaybackController` interface; `release()` removes its connection listener.
7. `PlaybackSessionHost` translates Media3 state into `PlaybackScreenModel` / `PlaybackScreenDependencies`; `PlayerScreen` consumes those UI-facing contracts rather than raw wiring helpers.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qianmokano/Asukaplayer](https://github.com/qianmokano/Asukaplayer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->
