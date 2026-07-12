---
trigger: always_on
description: **Updated:** 2026-06-22
---

# AGENTS.md - WorkoutTimer Flutter App

**Updated:** 2026-06-22
**Branch:** master

## OVERVIEW

Cross-platform Flutter workout rest timer with preset durations (30s/60s/90s/120s), multi-channel notifications, SQLite-backed workout history, AI plan generation, per-set recording, and bodyweight volume tracking. Supports Android, iOS, Web, and Desktop.

**Architecture**: MVVM with Provider (ChangeNotifier), services layer, local SQLite.
**Stack**: Flutter 3.10+ / Dart 3.10.7+ / sqflite + sqflite_common_ffi / provider / flutter_local_notifications / uuid / intl / fuzzy / fl_chart / cached_network_image / google_fonts / string_similarity / flutter_localizations (gen-l10n).
**Design System**: "Flat Vitality" — warm gradients, deep indigo accent (#1A237E), white circular buttons.
**Database**: SQLite v5 with incremental migrations (v1→v2→v3→v4→v5).

---

## COMMANDS

```bash
# Install & Run
flutter pub get                    # Install dependencies
flutter run                        # Run on device/emulator
flutter run -d chrome              # Web
flutter run -d windows             # Desktop

# Build
flutter build apk --debug          # Debug APK
./build_release.sh                 # Release APK (with --no-tree-shake-icons)
flutter build apk --release --no-tree-shake-icons  # Direct release build
flutter build web                  # Web build

# Install to phone (NEVER uninstall first — always overwrite)
adb install -r build/app/outputs/flutter-apk/app-debug.apk  # Overwrite install (preserves data)
# Do NOT use `flutter install` — it uninstalls first, wiping all user data

# Test
flutter test                                    # Run all unit tests
flutter test test/widget_test.dart              # Run single test file
flutter test test/services/exercise_matcher_service_test.dart  # Run specific test
flutter test test/services/database_migration_test.dart       # DB migration tests
flutter test --name "exact match"               # Run tests matching name
flutter test --reporter expanded                # Verbose output
flutter test integration_test/                  # Integration tests (ai_plan_import_e2e_test, detailed_recording_e2e_test)

# Analyze & Format
flutter analyze                    # Static analysis (all files)
flutter analyze lib/providers/      # Analyze specific directory
dart format lib/ test/             # Format code
dart fix --apply                   # Auto-fix issues

# Clean
flutter clean && flutter pub get   # Clean and reinstall
```

> **CRITICAL**: Always use `--no-tree-shake-icons` for release builds to prevent Material Icons from displaying as garbled text.

### CI
GitHub Actions on push/PR to `master`/`main`:
- `android-build.yml`: Java 17 → Flutter stable → `flutter pub get` → `flutter test` → `flutter build apk --debug`
- `ios-build.yml`: macOS runner, iOS build verification

### Linting
Uses `package:flutter_lints/flutter.yaml` — no custom rule overrides. Standard Flutter lint set.

---

## STRUCTURE

```
lib/
├── main.dart                 # Entry point, MultiProvider, bottom nav
├── animations/               # Page transitions and list animations
│   ├── animation_primitives.dart # AnimatedCard, CountUp, Shimmer
│   ├── list_animations.dart
│   └── page_transitions.dart # FadeUpPageRoute, ScaleFadePageRoute (slide-up transition)
├── providers/                   # State providers (ChangeNotifier, MVVM)
├── core/                        # ServiceLocator (dependency injection)
├── l10n/                        # Generated AppLocalizations + arb files
│   ├── timer_provider.dart   # Timer countdown, sets counter
│   ├── training_provider.dart # Training mode state machine
│   ├── plan_provider.dart    # Workout plan CRUD
│   ├── record_provider.dart  # History and stats
│   └── training_progress_provider.dart # Real-time training tracking
├── models/                   # Data models with fromMap/toMap
│   ├── workout_session.dart  # Simple session (sets, rest time)
│   ├── workout_record.dart   # Detailed record (exercises, weights)
│   ├── workout_plan.dart     # Plan template
│   ├── exercise.dart         # Exercise definition (870+ exercises)
│   ├── muscle_group.dart     # Muscle group enums + utilities
│   ├── set_data.dart         # Single set: setNumber, reps, weight
│   ├── calendar_plan.dart    # Date → plan mapping
│   ├── user_profile.dart     # AI plan profile: goal, frequency, equipment
│   └── weekly_plan_import.dart # JSON import for weekly plans
├── screens/                  # UI screens (full pages)
│   ├── timer_screen.dart     # Timer wrapper
│   ├── plan_screen.dart      # Workout plans + calendar
│   ├── plan_form_screen.dart # Plan creation/editing (924 lines)
│   ├── ai_plan_wizard_screen.dart # AI-powered plan generation
│   ├── ai_analysis_screen.dart    # AI analysis dashboard (1163 lines)
│   ├── exercise_selection_screen.dart # Exercise picker (811 lines)
│   ├── history_screen.dart   # Workout history list
│   ├── record_detail_screen.dart   # Detailed record view (832 lines)
│   ├── stats_screen.dart     # Statistics dashboard
│   ├── user_preferences_screen.dart # Training preferences (531 lines)
│   └── settings_screen.dart  # User preferences
├── widgets/                  # Reusable UI components
│   ├── training_widget.dart  # Main training UI

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kaiji-Z/workout-timer](https://github.com/Kaiji-Z/workout-timer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
