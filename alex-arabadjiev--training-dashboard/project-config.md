---
trigger: always_on
description: Android app that tracks a progressive daily exercise routine with adaptive goal setting. Exercise targets scale with a goal level (N) that adjusts based on daily performance:
---

# CLAUDE.md — Training Dashboard

## Project Overview

Android app that tracks a progressive daily exercise routine with adaptive goal setting. Exercise targets scale with a goal level (N) that adjusts based on daily performance:
- Goal level N: N push-ups, 2N sit-ups, 3N squats

**Goal level transitions** (evaluated each calendar day against the previous day's performance):
- 100% completion → N+1
- 33–99% completion → hold at N
- <33% completion → N-1 (floor: 1)

Progress is weighted by rep count: `completed reps / (N + 2N + 3N)`.

**Day N** (displayed prominently) = number of days where the user achieved ≥33% progress — not calendar days. This is computed from the Room database, no extra DataStore key needed.

The adaptive system is intentionally invisible to the user. There is no streak counter, no demotion messaging, no explanation of the rules. The user simply sees "Day N" and today's targets. Goals self-calibrate quietly.

Features: exercise completion tracking, adaptive goal progression, daily reminder notifications, settings dialog.

## Tech Stack

- **Language:** Kotlin (JVM target 17)
- **UI:** Jetpack Compose with Material 3
- **Architecture:** Single-Activity, MVVM (ViewModel + StateFlow)
- **Database:** Room (KSP code generation)
- **Preferences:** DataStore
- **Background work:** WorkManager (daily reminder notifications)
- **Build:** Gradle 8.5, Kotlin DSL, AGP 8.2.2, Kotlin 1.9.22
- **SDK:** minSdk 26, targetSdk 34, compileSdk 34

## Project Structure

```
app/src/main/java/com/example/trainingdashboard/
├── MainActivity.kt              # Single activity entry point
├── TrainingApp.kt               # Application class
├── data/
│   ├── PreferencesRepository.kt # DataStore wrapper (start date, reminder time)
│   └── db/
│       ├── AppDatabase.kt       # Room database (single instance)
│       ├── CompletionDao.kt     # DAO with Flow-based queries
│       └── DailyCompletion.kt   # Entity: composite key (dayNumber, exercise)
├── notification/
│   ├── ReminderScheduler.kt     # WorkManager periodic scheduling
│   └── ReminderWorker.kt        # Notification builder
├── ui/
│   ├── DashboardScreen.kt       # Main screen composable
│   ├── components/
│   │   ├── CompletionBanner.kt  # "All done!" banner with streak
│   │   ├── DayHeader.kt         # Day number display
│   │   └── ExerciseCard.kt      # Tap-to-complete exercise card
│   └── theme/
│       ├── Color.kt             # Material 3 color definitions
│       ├── Theme.kt             # Light/dark theme config
│       └── Type.kt              # Typography definitions
└── viewmodel/
    └── DashboardViewModel.kt    # UI state, day computation, streak logic
```

## Commands

- `./gradlew :app:assembleDebug` — build debug APK (output: `app/build/outputs/apk/debug/app-debug.apk`)
- `./gradlew :app:assembleRelease` — build signed release APK (requires `KEYSTORE_PASSWORD`, `KEY_ALIAS`, `KEY_PASSWORD` env vars)
- `./gradlew :app:testDebugUnitTest` — run unit tests
- `./gradlew :app:installDebug` — build + install on connected device/emulator

Use `./gradlew` (the wrapper, Gradle 8.5). The system `gradle` (4.4.1) is too old to build this project.

## Architecture Notes

- **Reactive data flow:** Room DAO returns `Flow<List<DailyCompletion>>` → ViewModel combines into `StateFlow<DashboardUiState>` → Compose collects state
- **Day calculation:** `ChronoUnit.DAYS` from stored start date to today; can be manually overridden via settings
- **Streak calculation:** Iterates backward from current day counting consecutive days where all 3 exercises are completed
- **Database design:** Composite primary key `(dayNumber, exercise)` with upsert for idempotent updates
- **Notifications:** `PeriodicWorkRequest` with 24h interval, `ExistingPeriodicWorkPolicy.UPDATE` for rescheduling

## Key Conventions

- All UI is in Jetpack Compose — no XML layouts
- Composable functions are stateless; state is managed in `DashboardViewModel`
- ProGuard keeps Room entity classes (`data.db.**`)
- Release signing uses environment variables: `KEYSTORE_PASSWORD`, `KEY_ALIAS`, `KEY_PASSWORD`

## CI/CD

- **CI** (`.github/workflows/ci.yml`): Runs on PRs to main — `assembleDebug` + `test`
- **Release** (`.github/workflows/release.yml`): Triggered by `v*` tags — builds signed APK, creates GitHub release

## Testing

Unit tests exist in `app/src/test/` covering ExerciseTargets, day computation, streak logic, count clamping, and FakeCompletionDao. Run with `./gradlew :app:testDebugUnitTest`.

Priority areas for additional tests:
- Room DAO: completion queries, upsert behavior (instrumented)
- UI: exercise card interactions, settings dialog (instrumented)
- Full ViewModel integration tests (requires Robolectric for AndroidViewModel)

## UI & Design

The app uses the **Kinetic design system** — a custom visual language documented in `.agents/context/DESIGN_GUIDE.md`. Read it before making any UI changes. Key rules summarised below; the guide is authoritative.

### Design Tokens
- All colors must use named `Kinetic*` tokens from `ui/theme/Color.kt` — never hardcode hex values or use raw `Color.Black` / `Color.White` for surfaces

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alex-arabadjiev/training-dashboard](https://github.com/alex-arabadjiev/training-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
