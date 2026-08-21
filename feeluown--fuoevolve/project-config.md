---
trigger: always_on
description: FuoEvolve is a Kotlin Multiplatform music player. The root Gradle project includes `:shared` and `:androidApp`; the Swift shell lives in `iosApp/`.
---

# Repository Guidelines

## Project Structure & Module Organization

FuoEvolve is a Kotlin Multiplatform music player. The root Gradle project includes `:shared` and `:androidApp`; the Swift shell lives in `iosApp/`.

The project is migrating from a flat shared source tree toward feature-oriented boundaries. Keep the current Kotlin package `org.feeluown.mobile` stable while this migration is in progress; physical source location now communicates ownership even when package declarations have not changed yet.

### Shared common sources

`shared/src/commonMain/kotlin/org/feeluown/mobile` is organized by responsibility:

- `app/`: app shell, `AppRoot`, typed navigation routes, app settings repository, and app-scoped state.
- `core/model/`: cross-feature contracts and models shared by multiple features.
- `core/ui/`: design system, theme, shared UI components, and common UI/platform abstractions.
- `feature/playback/`: playback orchestration, queue, lyrics, sleep timer, player UI, and the compatibility `FuoPlayerController` facade.
- `feature/search/`: search controller, immutable search state, search history, and search UI.
- `feature/provider/`: provider-facing feature controllers, auth/session boundaries, provider content UI, and capability interfaces.
- `feature/settings/`: settings state/controller/UI and resource-cache settings.
- `feature/localmusic/`: local-library state, refresh policy, controller, and UI.
- `feature/localplaylist/`: local-playlist repository/controller/state/UI.
- `feature/download/`: download state/controller/policies/UI.
- `feature/recognition/`: audio-recognition contracts, controller, and UI.
- `feature/home/`, `feature/onboarding/`, `feature/debug/`: feature-specific UI/state for those areas.
- `provider/`: provider protocol implementations and network adapters for Bilibili, NetEase Cloud Music, QQ Music, YouTube Music, and provider-core infrastructure.

Platform source sets remain under `shared/src/androidMain` and `shared/src/iosMain`. Shared multiplatform tests live in `shared/src/commonTest`.

### Platform applications

- `androidApp/src/main`: Android process/app shell, services, Media3 playback integration, platform repositories/stores, permissions, intents, and Android-specific composition root.
- `androidApp/src/main/kotlin/org/feeluown/mobile/AndroidAppContainer.kt`: Android dependency composition and process-scoped runtime wiring. `FuoEvolveApplication` should remain a thin host.
- `shared/src/iosMain/.../IosAppHost.kt`: iOS Kotlin host and `IosAppContainer`; platform dependency construction should stay in the container rather than leaking into common feature code.
- `iosApp/FuoEvolve`: Swift application shell and native Apple-platform integration.

## Architecture Rules

The intended dependency direction is:

`platform app/container -> app shell -> feature -> core contracts`

Provider/platform implementations sit behind interfaces consumed by features. Do not use a platform host or global singleton as a service locator from shared feature code.

### Feature ownership and state

Each feature should own its state and operations. Prefer immutable `UiState` exposed through `StateFlow`/Flow and actions/events flowing back into the owning controller or view model.

Do not add new app-global `isLoading`, `message`, or generic error flags to `FuoPlayerController`. Loading, errors, transient feedback, filters, selections, and request state should normally be feature-local. App startup state and truly global overlays are exceptions and should have an explicit app-level owner.

Compose `mutableStateOf` is appropriate for UI-local presentation state. Long-lived business/application state should prefer observable state holders with explicit ownership rather than being added directly to the global controller facade.

### Navigation ownership

Typed routes are defined in the app layer. `AppNavigator` / `FuoAppViewModel` are the intended owners of application navigation.

Feature controllers may request navigation through explicit callbacks/events, but should not independently maintain a second navigation stack or duplicate route state. Avoid introducing parallel `selectedX`, `lastX`, and route-payload state for the same destination; prefer typed route payloads plus feature-owned loading/state restoration.

### Player/controller boundaries

`FuoPlayerController` currently exists as a compatibility facade and application coordinator, but it must not continue growing into the public API for every feature.

New search, provider, settings, library, playlist, download, recognition, or other feature behavior should be implemented in the corresponding feature controller/state holder. Playback-specific functionality belongs under `feature/playback`.

When touching existing controller code, prefer moving ownership toward narrower collaborators instead of adding another delegated property or unrelated method to `FuoPlayerController`.

### Repository and provider boundaries


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [feeluown/FuoEvolve](https://github.com/feeluown/FuoEvolve) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
