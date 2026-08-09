---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Build the debug APK
./gradlew assembleDebug

# Install debug build on a connected device/emulator
./gradlew installDebug

# Build the release bundle (what CI runs)
./gradlew --no-configuration-cache bundleRelease

# Run all unit tests across every module
./gradlew test

# Run unit tests for a single module (e.g. core:data)
./gradlew :core:data:testDebugUnitTest

# Run a single test class/method within a module
./gradlew :core:data:testDebugUnitTest --tests "com.naveenapps.expensemanager.core.data.repository.CategoryRepositoryImplTest"
./gradlew :core:data:testDebugUnitTest --tests "*.CategoryRepositoryImplTest.someTestMethod"

# Run instrumented tests (needed for Room migration tests, which live in androidTest)
./gradlew :core:database:connectedDebugAndroidTest

# Unit test coverage (Jacoco) — per module, this is the task CI runs (`unit-test` job)
./gradlew debugCoverage
# Aggregated report across all modules, defined in the root build.gradle.kts
./gradlew allDebugCoverage

# Lint / format (Spotless + ktlint, applied to every .kt/.kts/.xml file)
./gradlew spotlessCheck
./gradlew spotlessApply
```

Room schema changes are hand-written migrations (see "Database" below) validated by instrumented tests under `core/database/src/androidTest/.../database/Migration<N>To<N+1>Test.kt` — these require `connectedDebugAndroidTest`, not the plain `test` task.

## Architecture

This is a multi-module Clean Architecture Android app (single-Activity, 100% Kotlin/Compose). The module graph enforces a strict one-way dependency direction; when adding code, respect these boundaries rather than reaching "up" a layer:

- **`app`** — application entry point, single Activity (`MainActivity`), Koin bootstrap, the root Compose `NavHost`, app-lock/biometric gating, in-app update check.
- **`core:model`** — plain domain models shared everywhere (no Android framework dependency).
- **`core:repository`** — repository *interfaces* only (e.g. `CategoryRepository`, `SettingsRepository`, `FeedbackRepository`). Feature and domain code depends on these, never on `core:data` directly.
- **`core:domain`** — use cases (one class per operation, `operator fun invoke()`), grouped by feature under `usecase/<feature>/`, each with a matching `di/<Feature>UseCaseModule.kt`.
- **`core:data`** — repository *implementations*, Koin DI module definitions (`di/KoinRepositoryModule.kt`, `di/KoinActivityModule.kt`), mappers between Room entities and domain models (`mappers/*Mappers.kt`).
- **`core:database`** — Room database, DAOs, entities, and all schema migrations (`DatabaseMigrations.kt`, `exportSchema = true`).
- **`core:datastore`** — Jetpack DataStore Preferences wrappers, one class per concern (e.g. `FeedbackDataStore`, `ThemeDataStore`), all backed by a single shared `DataStore<Preferences>` instance.
- **`core:navigation`** — the navigation abstraction described below (`AppComposeNavigator`, `ExpenseManagerScreens`).
- **`core:designsystem`** — shared Compose components/theme (`SettingRow`, `AppCardView`, `DeleteDialogItem`, `SettingsSection`, etc.) used by every feature module instead of hand-rolled UI.
- **`core:common`**, **`core:testing`**, **`core:notification`**, **`core:settings`** — shared utilities/extensions, test fixtures (`FakeConstants.kt`), notification scheduling, and a small settings-domain module (number formatting) respectively.
- **`feature:*`** (account, analysis, budget, category, country, currency, dashboard, export, filter, language, onboarding, reminder, settings, theme, transaction, about) — one module per user-facing feature. Each follows the same internal shape: `<Screen>.kt` (Compose), `<Screen>ViewModel.kt`, `<Screen>State.kt`, `<Screen>Action.kt`, optionally `<Screen>Event.kt`, and `di/<Feature>ViewModelModule.kt`.

Module wiring for a new feature module always applies the same convention plugins (defined in `build-logic/convention`): `naveenapps.plugin.android.feature`, `naveenapps.plugin.kotlin.basic`, `naveenapps.plugin.compose`, `naveenapps.plugin.di`. Other convention plugin ids: `naveenapps.plugin.android.library`, `naveenapps.plugin.android.app`, `naveenapps.plugin.room`. `MIN_SDK`/`TARGET_SDK`/`COMPILE_SDK` are defined once in `AndroidConfigExt.kt` and applied everywhere via these plugins — don't hardcode SDK versions in a module's own `build.gradle.kts`.

### Dependency injection (Koin)

DI is Koin, organized **per layer and per feature**, never one giant module:

- Each feature module exposes `di/<Feature>ViewModelModule.kt`; all of them are aggregated into one `ViewModelModule` in `app/.../di/ViewModelModule.kt` via `includes(...)`.
- Core layers each expose their own module (`RepositoryModule`, `UseCaseModule`, `DatabaseModule`, `DatastoreModule`, `NavigationModule`, `NotificationModule`, `AppModule`, `ActivityModule`).
- Koin itself is started from a Jetpack **App Startup** `Initializer` (`app/.../initializer/KoinInitializer.kt`), not from `Application.onCreate()`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nkuppan/expensemanager](https://github.com/nkuppan/expensemanager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
