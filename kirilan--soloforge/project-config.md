---
trigger: always_on
description: At the start of every new agent session, read `CLAUDE.md` and compare it with this file. If `CLAUDE.md` contains important project changes, conventions, feature status, branding decisions, migration rules, privacy constraints, or build/run notes that are missing or stale here, replicate the relevant updates into `AGENTS.md` before making code changes.
---

# Solo Forge — Project Context

## Session Startup

At the start of every new agent session, read `CLAUDE.md` and compare it with this file. If `CLAUDE.md` contains important project changes, conventions, feature status, branding decisions, migration rules, privacy constraints, or build/run notes that are missing or stale here, replicate the relevant updates into `AGENTS.md` before making code changes.

A **local-first Android fitness app**. No backend, no auth, no analytics, no cloud telemetry. The only outbound network call is the user's own OpenRouter API key for food-photo analysis.

## Features

1. **Intermittent fasting timer** — modes: 16:8, 18:6, 20:4, 36h. Smart context-aware reminders (no time-of-day spam):
   - "Almost there" encouragement 1h before fast ends
   - "Eating window closing" 1h before window ends, scheduled when a completed fast ends
   - Cancellation is automatic when the user takes the opposite action
2. **AI calorie counter** — user supplies their OpenRouter key, snaps a photo with optional comment, gets structured macros back, edits if needed, saves locally. Model choice is automatic through the escalation chain in `FoodAnalysisModels`.
3. **Weight tracking** — manual entries, line chart, edit/delete, weekly weigh-in reminder.
4. **Workout timer** — simple, interval, and exercise/rest timers with local workout logging and dashboard calorie bonus.
5. **Home dashboard** — at-a-glance tiles for fasting, today's nutrition vs. goals, weight, workout time, and streak.
6. **FOSS/privacy branding** — first-run intro and Settings/About emphasize GPL-3.0, no backend, no accounts, no analytics, and local-first data ownership without adding persistent dashboard clutter.

The full plan lives in `~/.claude/plans/i-want-to-make-spicy-crab.md` (outside the repo).

## Tech stack

- **Kotlin** + **Jetpack Compose** + **Material 3** (dynamic color)
- **Min SDK 26**, **compile/target SDK 35**
- **Hilt** for DI; **Room** for SQLite; **DataStore** for prefs; **EncryptedSharedPreferences** for the API key
- **WorkManager** for one-shot reminder workers; **foreground service** for the active-fast live notification
- **CameraX** for capture; **Ktor + kotlinx.serialization** for OpenRouter; **Coil** for image rendering
- **Vico** for charts (added but not yet used)
- **AGP 8.13.2**, **Gradle 9.0**, **Kotlin 2.0.21**, **KSP** (not kapt)

## Project layout

```
app/src/main/java/com/kbul/spicycrab/
├── MainActivity.kt                 // Single activity, hosts AppNav
├── SpicyCrabApp.kt                 // @HiltAndroidApp, creates notification channels
├── ui/
│   ├── theme/                      // Material 3 theme (Color, Type, Theme.kt)
│   ├── nav/AppNav.kt               // 5-tab bottom nav: Home, Fast, Food, Weight, Settings
│   ├── home/                       // Dashboard (currently placeholders)
│   ├── fasting/                    // FastingScreen + ProgressRing + ViewModel
│   ├── food/                       // List ↔ Capture ↔ Analyze; FoodViewModel
│   ├── weight/                     // Stub
│   └── settings/                   // SettingsScreen + ViewModel
├── data/
│   ├── db/                         // Room AppDatabase, entities, DAOs
│   ├── prefs/SettingsRepo.kt       // DataStore for goals, export URI, units
│   ├── prefs/SecureKeyStore.kt     // EncryptedSharedPreferences for API key
│   └── backup/BackupManager.kt     // Versioned JSON backup: export/import (merge or replace) + auto-backup
├── domain/
│   ├── fasting/                    // FastingMode, FastingRepository, StreakCalculator
│   └── nutrition/                  // FoodRepository, NutritionEstimate, ImageUtils
├── notifications/
│   ├── NotificationChannels.kt
│   ├── FastingNotificationService.kt   // Foreground service with 30s ticking
│   ├── ReminderScheduler.kt            // WorkManager scheduling
│   └── FastingReminderWorker.kt        // CoroutineWorker that posts notifications
├── network/
│   ├── OpenRouterClient.kt
│   ├── OpenRouterDtos.kt
│   └── VisionPrompts.kt            // System prompt + JSON schema
└── di/DatabaseModule.kt            // Hilt module for Room
```

## Conventions

- **Room migrations are mandatory.** `fallbackToDestructiveMigration()` is gone. To make a schema change:
  1. Edit the `@Entity`.
  2. Bump `version` in `AppDatabase.kt`.
  3. Add a `Migration(oldVersion, newVersion)` to `Migrations.kt` and append it to `ALL_MIGRATIONS`.
  4. Build once — KSP exports the new schema to `app/schemas/<n>.json` (commit it).
  5. Add a `MigrationTestHelper` test under `app/src/androidTest/...` that walks data through the new migration.
  6. **Notes**: SQLite ALTER TABLE ADD COLUMN with `NOT NULL DEFAULT x` produces a column with a recorded default that Room's schema check rejects unless the entity declares the same default. Use the rename-recreate-copy-drop pattern instead (see `MIGRATION_2_3` for a reference).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kirilan/SoloForge](https://github.com/kirilan/SoloForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
