---
trigger: always_on
description: AI-powered health tracker (Android) with region-aware diets & lifestyles. Log meals and
---

# FitBuddy — Agent Context

AI-powered health tracker (Android) with region-aware diets & lifestyles. Log meals and
workouts via **photo or loose text**; an LLM estimates calories/macros. Also has dashboards,
progress charts, editable meal review, and reusable food presets.

> Package: `com.anant.fitbuddy` · App name: **FitBuddy** (formerly "CalorieSamrat").
> The app was fully renamed — do not reintroduce the old name in code or UI.

## Tech stack
- **Kotlin** + **Jetpack Compose** (Material 3 / Material You dynamic color).
- **MVVM + Clean-ish architecture**: UI → ViewModel → Repository → (Room + Remote AI).
- **Room** for local persistence. **DataStore (Preferences)** for settings.
- **Retrofit + Moshi** (codegen adapters, `@JsonClass(generateAdapter = true)`) + OkHttp.
- **Manual DI** via `FitBuddyApp` (service locator) + `NetworkModule` (no Hilt/Dagger).
- Coroutines + Flow/StateFlow throughout.
- Custom Canvas charts (no external chart lib).

## Build / run
- **AGP 9.3.0**, **Kotlin 2.2.10**, **compileSdk 36.1**, **minSdk 29**, **targetSdk 36**.
  - Do NOT apply the `org.jetbrains.kotlin.android` plugin — AGP 9+ has built-in Kotlin support.
  - Do NOT add `navigation3` / `material3.adaptive` deps — they pull `lifecycle:2.11` which needs
    compileSdk ≥ 37 (not installed) and previously broke the build.
- JDK: OpenJDK 21 (`JAVA_HOME=/opt/homebrew/opt/openjdk@21/libexec/openjdk.jdk/Contents/Home`).
- Config cache is ON.
- **Device testing (smoke / local features / debugging):** always install and use the **debug** app
  (`com.anant.fitbuddy.debug`, launcher label **FitBuddy Dev**). Do **not** uninstall, overwrite, or
  test against the release app on the phone (`com.anant.fitbuddy`) — that install is the user's real
  app (different signing key); agents must never `adb uninstall` it or sideload a local release over
  it. See `.cursor/rules/debug-build-for-testing.mdc`.
- Commands (run from repo root):
  - Compile check: `./gradlew :app:compileDebugKotlin`
  - Debug APK (required for on-device agent work):
    `./gradlew :app:assembleDebug && adb install -r --user 0 app/build/outputs/apk/debug/app-debug.apk`
    then launch `com.anant.fitbuddy.debug/com.anant.fitbuddy.MainActivity`
  - Optimized release build: `./gradlew :app:assembleRelease` (R8 + resource shrink). Local
    `keystore.properties` must use a **local/dev** keystore (`fitbuddy-local.jks`), not the CI
    release key — see DISTRIBUTION.md. CI signs releases via GitHub `RELEASE_*` secrets only.
  - Install release over adb **only when the user explicitly asks** (personal profile only —
    never work profile / user 10):
    `adb install -r --user 0 app/build/outputs/apk/release/FitBuddy-*.apk`
  - `installDebug`/`installRelease` also pass `--user 0` via `android.installation.installOptions`.
    Wireless adb: prefer direct `adb install -r --user 0 <apk>` if Gradle's adb push hits
    EOF/broken pipe.

## Module / file map (`app/src/main/java/com/anant/fitbuddy/`)
- `FitBuddyApp.kt` — Application/service locator; builds `SettingsRepository` + `FitnessRepository`.
- `MainActivity.kt` — sets Compose content; reads `dynamicColor` before theming.
- `data/database/`
  - `Entities.kt` — `UserProfile`, `FoodLog`, `ExerciseLog`, `FoodPreset`.
  - `Daos.kt` — DAOs + result rows (`FoodDailySummary`, `ExerciseDailySummary`, `FoodTotals`).
  - `AppDatabase.kt` — Room DB **version 2**, `fallbackToDestructiveMigration(dropAllTables=true)`
    (schema changes wipe local data — fine for dev, add real migrations before shipping).
- `data/model/` — Moshi API models (`FitnessTrackerModels.kt`: `FitnessTrackerResponse`,
  `FoodAnalysis`, `Ingredient`, `Macros`, `ExerciseAnalysis`), plus domain models `FoodDraft`/
  `IngredientDraft` (editable meal; macros stored as per-100g rates for live rescaling) and `ModelOption`.
- `data/remote/`
  - `AiApi.kt` — Retrofit: `chatCompletion` (@Url + nullable Authorization), `listModels`
    (OpenRouter), `listGeminiModels` (@Url with `?key=`).
  - `NetworkModule.kt` — Moshi (codegen + reflective fallback), OkHttp, Retrofit (placeholder base URL; calls use @Url).
  - `RemoteAiDataSource.kt` — calls [PromptCatalog] for prompt text, image attach, JSON parse,
    `fetchFreeVisionModels` (OpenRouter, free+vision), `fetchGeminiVisionModels` (Gemini free
    Flash, ladder-ordered).
  - `dto/` — `ChatDtos.kt`, `ModelsDtos.kt` (OpenRouter `ModelDto` + Gemini `GeminiModelDto`).
- `data/prompts/` — `PromptCatalog.kt` loads LLM templates from
  `app/src/main/resources/prompts/` (`shared/` shells + `region/{india,us,europe,latin_america}/`
  overlays). Edit those `.txt` files to change prompts; do not re-embed prompt bodies in
  `RemoteAiDataSource` or `*RegionPack`.
- `data/region/` — `AppRegion`, `RegionPack` / `*RegionPack` (staples + UI hints; prompt overlays
  loaded via PromptCatalog), `RegionDetector`.
- `data/repository/`
  - `FitnessRepository.kt` — single `analyze()` entry point; routes by response `status`
    (SUCCESS→FoodReady draft, EXERCISE_LOGGED→save, CLARIFICATION_REQUIRED→ask); offline
    `simulateAIService` fallback; presets CRUD; `getFoodTotalsToday` (single consolidated query).
  - `AnalysisOutcome.kt` — sealed: `FoodReady(draft)`, `ExerciseSaved`, `NeedsClarification`, `Error`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anantdark/FitBuddy](https://github.com/anantdark/FitBuddy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
