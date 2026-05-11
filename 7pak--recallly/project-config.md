---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Recallly is a B2B, offline-first, AI-powered native Android app for field professionals (Sales Reps, Field Engineers, Insurance Adjusters). It eliminates manual CRM data entry using on-device AI and system integrations.

- **Package**: `com.at.recallly`
- **Min SDK**: 28 (Android 9) / **Target SDK**: 36 / **Compile SDK**: 36
- **JVM Target**: Java 11
- **Compose BOM**: 2026.02.01
- **AGP**: 8.13.2 / **Gradle**: 8.13 / **Kotlin**: 2.3.10

## Build Commands

```bash
./gradlew build              # Full build
./gradlew assembleDebug      # Build debug APK
./gradlew assembleRelease    # Build release APK
./gradlew test               # Run unit tests
./gradlew connectedAndroidTest  # Run instrumented tests (requires device/emulator)
./gradlew :app:testDebugUnitTest --tests "com.at.recallly.ExampleUnitTest"  # Single test class
./gradlew clean              # Clean build artifacts
```

On Windows, use `gradlew.bat` instead of `./gradlew`.

## Secrets & Local Configuration

`local.properties` (gitignored) must contain these keys, which get injected as `BuildConfig` fields:

- `WEB_CLIENT_ID` — Firebase/Google Sign-In OAuth client ID
- `GEMINI_API_KEY` — Gemini AI API key for field extraction
- `ADMOB_APP_ID` — AdMob application ID (injected via manifest placeholder; defaults to test ID)
- `ADMOB_REWARDED_PRE_RECORD_ID` — Rewarded ad unit for pre-record (defaults to test ID)
- `ADMOB_REWARDED_POST_SAVE_ID` — Rewarded ad unit for post-save (defaults to test ID)
- `BILLING_SUBSCRIPTION_ID` — Play Billing subscription product ID (defaults to `recallly_premium_monthly`)

These are read at build time via `Properties` in `app/build.gradle.kts`.

## Architecture

**Clean Architecture + MVVM** with Unidirectional Data Flow (UDF). Single `:app` module.

### Layer Structure (under `com.at.recallly`)

- **`presentation/`** — UI layer (Compose screens, ViewModels, Navigation)
  - ViewModels expose exactly ONE `StateFlow<UiState>` and accept `UiEvent` sealed interfaces
  - No business logic in ViewModels
  - Feature screens organized by feature (e.g., `auth/` has LoginScreen, SignUpScreen, AuthViewModel, AuthUiState, AuthUiEvent)
- **`domain/`** — Business logic (pure Kotlin models, repository interfaces, UseCases)
  - No Android dependencies allowed here
  - UseCases use `operator fun invoke()` pattern
- **`data/`** — Data layer (Room DAOs, DataStore, repository implementations, API clients)
- **`core/`** — Shared infrastructure
  - `di/` — Koin dependency injection modules
  - `result/` — `Result<T>` sealed class (Success/Error) for error handling
  - `theme/` — Material3 Color, Theme, Typography
  - `util/` — DispatcherProvider, Constants, Extensions, RecalllyException

### Navigation Flow

`Splash → (Login/SignUp) → Language Selection → Persona → Fields → WorkSchedule → DataConsent → Home`

Routes defined as `@Serializable` sealed classes in `presentation/navigation/Route.kt`. The `RecalllyNavGraph` observes auth + onboarding state via `LaunchedEffect` to auto-navigate. Settings has sub-routes with parameter passing (e.g., `SettingsFieldSelection(fromPersonaChange: Boolean)`).

### Key Files

- **Entry point**: `MainActivity.kt` → `RecalllyNavGraph` → screens
- **Application**: `RecalllyApplication.kt` (Koin + Timber initialization, loads saved language via `runBlocking`)
- **DI**: `core/di/AppModule.kt` (single Koin module — all registrations here)
- **Navigation**: `presentation/navigation/RecalllyNavGraph.kt` + `Route.kt`
- **Database**: `data/local/db/RecalllyDatabase.kt` (Room, schema exports to `app/schemas/`)
- **Preferences**: `data/local/datastore/PreferencesManager.kt`
- **Voice note storage**: `data/local/file/VoiceNoteFileStorage.kt` — JSON file at `filesDir/voice_notes.json`, Mutex for thread safety
- **PDF export**: `data/export/PdfExportService.kt` — native Android `PdfDocument` API, A4 pages, uses `FieldLocalizer` for localized field names
- **Language**: `core/util/LanguageManager.kt` — manages locale switching via `AppCompatDelegate.setApplicationLocales`
- **Ads**: `data/ad/RewardedAdManager.kt` — Google Mobile Ads rewarded ads (initialized in `RecalllyApplication`)
- **Billing**: `data/billing/BillingClientWrapper.kt` + `PremiumPreferences.kt` — Google Play Billing for premium subscription
- **Reminders**: `data/notification/AlarmReminderScheduler.kt` — `AlarmManager`-based reminders with `ReminderReceiver` + `BootReceiver` for persistence across reboots

## Speech Recognition (Dual-Mode)

The app uses two speech recognition paths, chosen at mic-tap time via `ConnectivityChecker`:

- **Online**: Android `SpeechRecognizer` with `RecognitionListener` (real-time partial results)
- **Offline**: whisper.cpp via JNI (batch transcription after recording finishes)

### whisper.cpp Native Integration

- **Git submodule** at `app/src/main/cpp/whisper.cpp/` (from `ggml-org/whisper.cpp`)
- **CMake** build at `app/src/main/cpp/CMakeLists.txt`, JNI bridge in `app/src/main/cpp/jni.c`
- **NDK**: r26.3 (`26.3.11579264`) — required for 16KB page alignment (Android 15+ Play Store requirement)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [7pak/Recallly](https://github.com/7pak/Recallly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
