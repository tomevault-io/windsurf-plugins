---
trigger: always_on
description: This file provides guidance to WARP (warp.dev) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

Komi Store is a cross-platform app store for GitHub releases built with **Kotlin Multiplatform (KMP)** and **Compose Multiplatform**. It targets **Android** (min API 26, target 36) and **Desktop** (Windows, macOS, Linux via JVM).

Package: `zed.rainxch.githubstore`

## Build & Run Commands

```bash
# Android debug build
./gradlew :composeApp:assembleDebug

# Desktop (run in dev mode)
./gradlew :composeApp:run

# Full build check (both platforms)
./gradlew build

# Lint (ktlint auto-formats on preBuild/compileKotlin* tasks automatically)
./gradlew ktlintFormat           # manual format all modules
./gradlew ktlintCheck            # check without fixing

# Desktop installers
./gradlew :composeApp:packageDmg    # macOS
./gradlew :composeApp:packageExe    # Windows
./gradlew :composeApp:packageDeb    # Linux
```

**Requirements:** JDK 21+ (Temurin recommended), Android SDK for Android builds.

**Setup:** Create a GitHub OAuth App and put `GITHUB_CLIENT_ID=<your_id>` in `local.properties` (root). Callback URL: `githubstore://callback`.

## Architecture

**Clean Architecture + MVVM** with strict layer separation:

- **Domain** — Repository interfaces, models, use cases. No framework dependencies.
- **Data** — Repository implementations, Ktor API clients, Room DAOs, DTOs, mappers. Each feature's DI module lives in `data/di/SharedModule.kt`.
- **Presentation** — ViewModels with `StateFlow`/`Channel`, Compose screens.

### State Management Pattern (every screen)

Every ViewModel follows the same State/Action/Event pattern:
- `State` — data class holding all UI state, exposed via `StateFlow`
- `Action` — sealed interface for user input (clicks, refreshes)
- `Event` — sealed interface for one-off effects (navigation, toasts), sent via `Channel.receiveAsFlow()`

### Module Layout

```
composeApp/          # App entry points, navigation, DI wiring
  src/commonMain/    # Shared UI & wiring
  src/androidMain/   # Android entry (MainActivity)
  src/jvmMain/       # Desktop entry (DesktopApp.kt)
core/
  domain/            # Shared interfaces, models, use cases
  data/              # Networking (Ktor), database (Room), DI, platform impls
  presentation/      # Material 3 theming, reusable UI components, localized strings (13 languages)
feature/<name>/
  domain/            # Feature-specific interfaces & models
  data/              # Feature-specific implementations & Koin DI module
  presentation/      # Feature ViewModel + Compose screens
build-logic/convention/  # Custom Gradle convention plugins
```

Some features (favourites, starred, recently-viewed, tweaks) are **presentation-only** — they use core repositories directly and register ViewModels in `composeApp/.../di/ViewModelsModule.kt` instead of having a `data/di/` layer.

### Convention Plugins (build-logic)

| Plugin ID | Use For |
|-----------|---------|
| `convention.kmp.library` | KMP shared library modules (domain, data) |
| `convention.cmp.library` | Compose Multiplatform library modules |
| `convention.cmp.feature` | Feature presentation modules (auto-adds Compose + Koin + core:presentation) |
| `convention.cmp.application` | Main app module |
| `convention.room` | Room database modules |
| `convention.buildkonfig` | Build-time config (reads from local.properties) |

### Navigation

Type-safe navigation using `@Serializable` sealed interface `GithubStoreGraph` in `composeApp/.../navigation/GithubStoreGraph.kt`. Routes are wired in `AppNavigation.kt`. Parameterized routes: `DetailsScreen(repositoryId, owner, repo, isComingFromUpdate)`, `DeveloperProfileScreen(username)`.

### Dependency Injection

**Koin** — each feature's data layer defines a module in `data/di/SharedModule.kt`. All modules are registered in `composeApp/.../di/initKoin.kt`. ViewModels injected via `koinViewModel()`. `DetailsViewModel` and `MirrorPickerViewModel` use manual Koin `viewModel { }` with `parametersOf()` for constructor args; all others use `viewModelOf(::ClassName)`.

### Key Cross-Cutting Concerns

- **Auth flow:** GitHub device-flow OAuth. Primary path goes through backend proxy (`/v1/auth/device/start`, `/v1/auth/device/poll`); falls back to direct GitHub only on infrastructure errors (5xx, timeouts). HTTP 4xx and GitHub's negative 200-bodies never trigger fallback. Backend rate limits (10 starts/hr, 200 polls/hr per IP) are hard — do not add retry loops.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kurikomi-labs/komi-store](https://github.com/kurikomi-labs/komi-store) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
