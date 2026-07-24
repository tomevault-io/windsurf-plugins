---
trigger: always_on
description: Kotlin Multiplatform (KMP) app using **Compose Multiplatform** targeting Android and iOS. Displays movie data from The Movie Database (TMDB) API.
---

# MovieDB-App

## Project Overview

Kotlin Multiplatform (KMP) app using **Compose Multiplatform** targeting Android and iOS. Displays movie data from The Movie Database (TMDB) API.

- **Kotlin:** 2.3.20
- **Compose Multiplatform:** 1.10.3
- **AGP:** 9.1.0
- **Gradle:** 9.4.1
- **Min SDK:** 28 | **Target/Compile SDK:** 36
- **Java / JVM Target:** 21
- **Package:** `com.gabrielbmoro.moviedb`

## AI Context — Read This First

This file is the single source of truth for all AI coding assistants. Before starting any task:

1. **Skills** — List files in `ai/skills/`. Identify which skill covers the task at hand, read that skill file in full before proceeding.
2. **Platform context** — For Android-specific tasks, read `ai/instructions/android.md`. For iOS-specific tasks, read `ai/instructions/ios.md`.
3. **Module graph** — `ai/module-graph.md` defines all module dependency constraints. Any code change must respect these rules.

## Build Commands

All projects live under `src/`. Run commands from the `src/` directory.

| Command | Description |
|---|---|
| `./gradlew composeApp:compileKotlinDesktop` | Quick Kotlin compilation check |
| `./gradlew :composeApp:connectedCheck` | Run instrumentation tests |
| `./gradlew detektAll` | Run Detekt linting |
| `./gradlew koverHtmlReportAll` | Generate Kover coverage report |
| `./gradlew :build-logic:checkPopcornGuineapig` | Verify module dependency rules |
| `./gradlew build` | Full build |

## Architecture: Clean Architecture + MVI

### Layer Dependency Rules (enforced by Popcorn Guineapig)

```
composeApp (UI orchestrator)
    |
    ├── feature:*      → depends on domain, designsystem, platform
    ├── domain         → LEAF — no project module dependencies
    ├── data           → depends ONLY on domain (implements repository interfaces)
    ├── designsystem   → LEAF — no project module dependencies
    ├── platform       → LEAF — no project module dependencies
```

### Data Flow

```
UI (Screen composable)
  → ViewModel (intent → state via StateFlow)
    → UseCase (business logic)
      → Repository interface (in domain)
        → MoviesDataRepository (in data)
          → ApiService (Ktor → TMDB API)
          → FavoriteMoviesDAO (Room → SQLite)
```

## Tech Stack

| Category | Library | Version |
|---|---|---|
| UI | Compose Multiplatform + Material3 | 1.10.3 / 1.9.0 |
| Compose Compiler Plugin | Bundled with Kotlin compiler | 2.3.20 |
| Navigation | Jetpack Navigation Compose | 2.9.2 |
| Lifecycle | lifecycle-viewmodel-compose | 2.10.0 |
| Networking | Ktor (OkHttp/Darwin engines) | 3.4.2 |
| Serialization | kotlinx-serialization | — |
| Image Loading | Coil 3 (ktor3 network) | 3.4.0 |
| Database | Room + sqlite-bundled | 2.8.4 / 2.6.2 |
| DI | Koin (annotations + KSP + Koin Compiler) | 4.2.1 |
| Koin Annotations | KSP compiler for `@Module`/`@Factory`/`@Single` | 2.3.1 |
| Koin Compiler Plugin | Custom convention plugin for Koin compiler | 1.0.0 |
| State | Coroutines + StateFlow | 1.10.2 |
| Collections | kotlinx-collections-immutable | 0.4.0 |
| Logging | Kermit | 2.1.0 |
| Deep Links | Rinku | 1.6.0 |
| Build Config | BuildKonfig | 0.18.0 |
| Dependency Audit | Popcorn Guineapig | 3.2.0 |
| Linting | Detekt | 1.23.8 |
| Coverage | Kover | 0.9.8 |
| Crash Reporting | Firebase Crashlytics | — |
| Analytics | Kotzilla | 2.0.8 |
| CI | Bitrise + GitHub Actions | — |

Dependency versions are sourced from `src/gradle/libs.versions.toml`.

## Module Structure

```
src/
├── composeApp/          # NavHost, DI aggregator, RootApp.kt
├── data/                # ApiService, DTOs, DAOs, DatabaseProvider, Mappers
├── domain/              # Domain models, Repository interfaces, UseCases
├── designsystem/        # Theme, Colors, shared UI (cards, toolbars, icons, AsyncImage)
├── platform/            # Navigation, PagingController, BaseViewModel, Logging, VideoPlayer
├── feature-movies/      # Movie grid with filter tabs + pagination
├── feature-details/     # Movie detail (backdrop, rating, favorite, info)
├── feature-search/      # Debounced search with results
├── feature-wishlist/    # Favorites list with swipe-to-delete
├── androidApp/          # Android entry (Application, MainActivity)
├── iosApp/              # Xcode project
└── build-logic/         # Convention plugins (kmp-library, koin-compiler, popcorngp)
```

## Navigation

- **Type:** Jetpack Navigation Compose Multiplatform
- **Routes** (`Screen` enum in `platform` module): `Movies`, `Details/{movieId}`, `Search`, `Wishlist`
- **NavHostController** exposed via `CompositionLocal` (`LocalNavController`)
- **Deep links:** Rinku library handles `movie/{id}`, `search?query=`, `favorite` URIs
- Navigation graph built in `RootApp.kt` using extension functions from `NavHostGraphBuilderExt.kt`

## State Management (MVI)

Each feature screen follows the **Model-View-Intent** pattern via `BaseViewModel`:

```kotlin
abstract class BaseViewModel<State : UiState, Intent : UserIntent, Event : UiEvent>
```

- **State:** data class implementing `UiState` interface
- **Intent:** sealed class/interface implementing `UserIntent`
- **Event:** sealed class/interface implementing `UiEvent`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gabrielbmoro/MovieDB-App](https://github.com/gabrielbmoro/MovieDB-App) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
