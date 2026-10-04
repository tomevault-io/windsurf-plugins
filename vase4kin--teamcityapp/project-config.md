---
trigger: always_on
description: Use this file as the working contract for AI coding agents in this repository.
---

# TeamCityApp Agent Guide

Use this file as the working contract for AI coding agents in this repository.
The target architecture follows the modern approach described in the Now in
Android agent guide. Use it by default for every new feature and whenever
refactoring existing UI, state, data, dependency injection, or models. The current
codebase has not completed that migration. Check nearby code and module build
files before editing, preserve unrelated local work, and keep each change scoped
to the requested behavior and the coherent migration slice needed to support it.

## Project snapshot

- TeamCityApp is a native Android client for JetBrains TeamCity.
- The app currently uses multiple activities and fragments, XML layouts, Data Binding,
  presenters and legacy state holders, RxJava 2, Hilt for Android screen injection,
  a separate Dagger account API graph, Retrofit/Gson,
  RxCache, and SharedPreferences. Java and Kotlin coexist.
- About uses Compose, Material 3, a Hilt ViewModel, and StateFlow. Its suspend
  repository adapts the existing Rx API and cache. Lifecycle-aware state collection
  starts loading; losing the last collector cancels pending work immediately,
  while completed content survives configuration changes. About is split into
  `features/about/api` and `features/about/impl`, with Kotlin source roots.
  The app retains the legacy server DTO and its serialized class name for RxCache
  compatibility; the app repository maps it into About’s plain API model. About UI tests
  run in the app instrumentation suite used by Marathon. The licenses action
  still opens Google's OSS licenses screen.
- Other screens remain legacy. Room, DataStore, WorkManager, and Navigation 3
  remain target technologies.
- Builds use Kotlin DSL Gradle files, a version catalog, type-safe project accessors,
  an included `build-logic` build for conventions and SDK/application settings,
  and JDK 17.
- The app has `mock` and `prod` flavors, and `debug` and `release` build types.
  `mockRelease` is disabled.

## Repository layout

- `app/`: application, TeamCity API and cache integration, shared app flows, and
  features that have not been moved to standalone modules.
- `features/`: feature modules. About uses `api`/`impl`. Unmigrated features
  still use `models`/`repository`/`feature` or single-module layouts; migrate the
  owning feature to `api`/`impl` during substantive refactors.
- `libraries/`: shared API, storage, models, resources, theme, networking helpers,
  security, utilities, and other reusable components.
- `build-logic/src/main/kotlin/Config.kt`: SDK/JVM settings and application version.
- `build-logic/`: included Kotlin build containing Android application/library,
  Java-only library, Hilt/kapt, Compose, Data Binding, and aggregate coverage conventions.
  All Android modules apply these conventions; `buildSrc` has been retired.
- `gradle/libs.versions.toml`: dependency and plugin versions/coordinates.
- `settings.gradle.kts`: included modules, plugin resolution, and dependency
  repositories. Root and module `build.gradle.kts` files use Kotlin DSL.
- `.github/workflows/build.yml`: build, lint, unit test, instrumentation, and
  coverage CI. Keep GitHub Actions as the CI platform. `scripts/` contains R8
  verification preparation/documentation.

## Target architecture

- New features and substantive refactors must use this architecture whenever
  feasible. Existing legacy patterns are compatibility constraints, not the
  default for new code. If a concrete constraint prevents adoption, explain it
  and keep the legacy dependency behind a small, explicit adapter.
- UI: use Jetpack Compose, Material 3, shared design-system components, and
  adaptive layouts. Use window size classes for window-level navigation/pane
  decisions and local layout constraints for responsive components. The destination
  is a single-activity app with
  Navigation 3. Use Navigation 3 for new navigation infrastructure; preserve
  existing activity/fragment entry points through adapters during screen migration.
- State: use Jetpack ViewModels, Coroutines, and `Flow`/`StateFlow` with
  unidirectional data flow. State-changing and business events go to ViewModels;
  immutable UI state flows back to the UI. UI-only interactions, navigation
  execution, and platform launches belong in the UI/navigation layer. When
  business state determines navigation, expose that state from the ViewModel
  and execute navigation in the UI. Model applicable loading, empty, error, and
  content states explicitly, including failures of optional content sections.
  Keep Activities, Fragments, views, adapters, and UI callbacks out of ViewModels.
- Follow the Now in Android Bookmarks pattern for migrated screens: a route/container
  composable obtains its Hilt ViewModel with `hiltViewModel()`, collects state with
  `collectAsStateWithLifecycle()`, and passes immutable state and callbacks to a
  separate stateless screen/content composable. Activities host the route.
- Derive UI state from repository flows with operators such as `map`, `onStart`,
  `catch`, and `stateIn(viewModelScope, SharingStarted.WhileSubscribed(...), ...)`.
  Prefer collection-driven loading over Activity calls to ViewModel `start`/`stop`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vase4kin/TeamCityApp](https://github.com/vase4kin/TeamCityApp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
