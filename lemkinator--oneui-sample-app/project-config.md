---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```powershell
./gradlew assembleDebug      # build debug APK
./gradlew assembleRelease    # build release APK (falls back to debug signing if no signing props)
./gradlew build              # full build (used in CI)
```

Test suite: unit tests (Kotest/MockK), Robolectric integration tests, Roborazzi screenshot tests, Kover coverage. Run with
`./gradlew testDebugUnitTest`.

Instrumented tests run via Gradle Managed Device (no physical device needed):

```powershell
./gradlew pixel9Api35DebugAndroidTest   # downloads ~1 GB image on first run, cached after
```

The GMD device (`pixel9Api35`: Pixel 9 / API 35 / aosp / x86_64) is declared once in root
`build.gradle.kts` and shared by `:app` instrumented tests and `:benchmarks` baseline profile generation.

## Private Dependencies (Required for Build)

`oneui-design` is hosted on a private GitHub Maven repo. Provide `ghUsername` + `ghAccessToken` (`read:packages` scope) via **one** of
these (checked in order):

1. `github.properties` in project root: `ghUsername=...` / `ghAccessToken=...`
2. `~/.gradle/gradle.properties`: `ghUsername=...` / `ghAccessToken=...`
3. Env vars: `GH_USERNAME` / `GH_ACCESS_TOKEN`

Release signing properties (`releaseStoreFile`, `releaseStorePassword`, `releaseKeyAlias`, `releaseKeyPassword`) use the same lookup order.

## Architecture

Single-module (`:app`) Android app demonstrating OneUI-Design components. Layered architecture (data/domain/ui):

- **`data/`** - `UserSettingsRepository`: SharedPreferences-backed store for user settings. Exposes per-field property delegates and a
  `StateFlow<UserSettings>`. Multi-field batch updates via `userSettings.update { copy(field = value) }` (synchronized). Single-field writes
  assign directly: `userSettings.search = "query"`.
- **`domain/`** - Use cases with `operator fun invoke()`. Suspend use cases (e.g. `CompleteOnboardingUseCase`) switch to
  `Dispatchers.Default`; flow-based use cases (e.g. `ObserveIconListUseCase`) return a `Flow` directly.
- **`ui/`** - Activities for settings/about/OOBE/pickers; Fragments for main tabs (`TabDesign`, `TabIcons`, `TabPicker`) with nested subtabs
  via ViewPager2. Screens with non-trivial async state use ViewModels (`AboutViewModel`, `SettingsViewModel`, `OOBEViewModel`,
  `SwitchBarViewModel`, `AppPickerViewModel`); simpler screens inject use cases or the repository directly.
- **`App.kt`** - `@HiltAndroidApp` entry point; `PersistenceModule.kt` - Hilt singleton providing `SharedPreferences` and
  `UserSettingsRepository` (with an `@ApplicationScope` `CoroutineScope` for the `StateFlow`).

State collection patterns:

- Fragments: `launchAndRepeatWithViewLifecycle { flow.collectLatest { } }`, or the `collectState` / `collectEvents` shortcuts from
  `ui/util/LifecycleUtils.kt`
- Activities: `collectState(viewModel.state) { }` / `collectEvents(viewModel.events) { }`
- One-shot delayed UI work: `viewLifecycleOwner.lifecycleScope.launch { delay(ms); ... }` (canceled automatically on `onDestroyView`)

ViewBinding uses the `autoCleared` delegate (`ui/util/AutoClearedUtils.kt`) to prevent leaks.

## Robolectric + JUnit 5

**Do not migrate Robolectric tests to JUnit 5.** `org.robolectric.junit.jupiter.RobolectricExtension` does not exist - Robolectric has no
native JUnit 5 support ([issue #3477](https://github.com/robolectric/robolectric/issues/3477)). The community extension
`tech.apter.junit5.jupiter:robolectric-extension` only targets Robolectric 4.14.1, is pre-release, and has no Hilt/Roborazzi support.

`@RunWith(RobolectricTestRunner::class)` + `junit-vintage-engine` is correct. Keep until Robolectric ships native JUnit 5.

**Kover + inline functions**: JUnit 4 + `RobolectricTestRunner` enables JaCoCo SMAP attribution — inlined call-site coverage is mapped back to the original `inline fun` definition. Simple delegating `inline fun` therefore don't need `@NoCoverage` here (tests calling them cover the definition via SMAP). The exception is `crossinline` default-value lambdas: the default compiles to a definition-site private static method that is never invoked, so it always needs `@NoCoverage` regardless of test runner. That is why the `@NoCoverage` footprint here is smaller than in `common-utils` (which uses JUnit 5 + `RobolectricExtension`, where SMAP attribution doesn't fire).

## Dependency Version Policy

Default: use the latest stable version of every dependency.
Renovate keeps minor/patch current; bump majors manually with release-note review.

Known exceptions:

1. Kotlin + KSP lockstep - Renovate's `kotlin` group enforces this
2. Detekt on fresh Kotlin majors. May need alpha until stable catches up
3. Plugin AGP compatibility windows, check before bumping AGP
4. CI emulator images, pin to most stable, not newest

## Static Analysis

Four tools run as part of `./gradlew build`:

- **Spotless** - enforces formatting via ktlint (sole ktlint driver; Detekt has no ktlint wrapper). Fix violations with
  `./gradlew spotlessApply`.
- **Detekt** - static analysis; config at `config/detekt/detekt.yml`. `autoCorrect = false`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lemkinator/OneUI-Sample-App](https://github.com/Lemkinator/OneUI-Sample-App) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
