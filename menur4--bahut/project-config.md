---
trigger: always_on
description: **Bahut** is a Flutter mobile app that connects to the **École Directe** API (French education platform) to let students visualize their grades, schedule, homework, and school behavior.
---

# Bahut — Claude Code Context

## Project Overview

**Bahut** is a Flutter mobile app that connects to the **École Directe** API (French education platform) to let students visualize their grades, schedule, homework, and school behavior.

- **Package:** `com.frhamon.calcul_moyenne`
- **Version:** 1.4.0+9
- **Platforms:** iOS 14+, Android SDK 21+, Web, macOS, Linux, Windows
- **Language:** French throughout (UI strings, comments, commit messages)
- **Dart SDK:** ^3.10.7

---

## Architecture

Feature-based modular architecture with clean separation:

```
lib/
├── core/               # Shared infrastructure
│   ├── accessibility/  # HIG compliance, haptic feedback, min touch targets
│   ├── constants/      # API endpoints, app constants
│   ├── demo/           # Demo mode data (Play Store: demo@bahut.app / demo2024)
│   ├── errors/         # Freezed union Failure type
│   ├── network/        # Dio API client with cookie/token management
│   ├── services/       # background sync, notifications, calendar, widgets, etc.
│   └── theme/          # 7 color palettes + auto-switching by time
├── features/           # 16 feature modules (each has data/domain/presentation)
├── shared/             # Shared widgets, animations, navigation
├── router/             # GoRouter config
├── app.dart            # Root widget
└── main.dart           # Entry point + service initialization
```

Each feature follows: `data/models/` → `presentation/providers/` → `presentation/screens/` → `presentation/widgets/`.

---

## State Management

**Flutter Riverpod 2.x** with `StateNotifierProvider`. Key providers:

| Provider | File | Purpose |
|---|---|---|
| `authStateProvider` | `features/auth/presentation/providers/auth_provider.dart` | Auth, MFA, biometric, multi-account |
| `gradesStateProvider` | `features/grades/presentation/providers/grades_provider.dart` | Grades, filtering, weighted averages |
| `homeworkStateProvider` | `features/homework/presentation/providers/homework_provider.dart` | Homework list + status |
| `scheduleStateProvider` | `features/schedule/presentation/providers/schedule_provider.dart` | Schedule data |
| `statisticsProvider` | `features/statistics/presentation/providers/statistics_provider.dart` | Computed analytics |
| `badgesProvider` | `features/gamification/presentation/providers/badges_provider.dart` | Badge unlock logic |
| `themeModeProvider` | `core/theme/theme_provider.dart` | Light/dark/system |
| `appThemeTypeProvider` | `core/theme/theme_provider.dart` | Color palette selection |
| `autoThemeConfigProvider` | `core/theme/theme_provider.dart` | Time-based auto-switching |

---

## Navigation

**GoRouter v14.6.2** — `router/app_router.dart`

Auth redirect logic: loading → login → qcm (MFA) → biometric → children (multi-account) → dashboard

ShellRoute with bottom nav (5 tabs: Dashboard, Grades, Schedule, Homework, Profile):
- `/dashboard`, `/grades`, `/schedule`, `/homework`, `/vie-scolaire`
- `/averages`, `/profile`, `/settings`, `/statistics`, `/simulation`, `/badges`

---

## API — École Directe

**Base URL:** `https://api.ecoledirecte.com/v3/`

Auth flow:
1. GET GTK cookie from `/login.awp?gtk=1`
2. POST credentials → receive `X-Token` JWT header
3. All subsequent requests include `X-Token`

Key endpoints:
- `/v3/login.awp` — Authentication
- `/v3/eleves/{id}/notes.awp` — Grades
- `/v3/eleves/{id}/vieScolaire.awp` — School behavior
- `/v3/E/{id}/emploidutemps.awp` — Schedule
- `/v3/Eleves/{id}/cahierdetexte.awp` — Homework

API client: `core/network/api_client.dart` (Dio + cookie jar + token interceptor)

---

## Data Models

All models use **@freezed** annotation (immutable, copyWith, JSON serialization, pattern matching).

- `GradeModel` — with extensions: `valeurDouble`, `valeurSur20`, `isValidForCalculation`, `mainSubjectName`
- `UserModel` / `ChildModel`
- `HomeworkModel` / `CahierDeTexteJour`
- `ScheduleModel`
- `VieScolaireModel`
- `Failure` (union: server / network / auth / cache / biometric / unknown)

Generated files (`*.freezed.dart`, `*.g.dart`) — run `flutter pub run build_runner build` after model changes.

---

## Key Features

1. **Multi-account** — Switch between student accounts; biometric unlock
2. **Grades** — Weighted averages, period filtering, class comparisons, color-coded indicators
3. **Statistics** — fl_chart visualizations, 30-day trends, goal tracking
4. **Gamification** — 15+ badges with celebration animations
5. **Schedule** — Calendar sync with device (device_calendar)
6. **Homework** — Due dates, completion status
7. **Home screen widgets** (Android) — Average, homework, schedule widgets (Kotlin: `HomeworkWidgetProvider.kt`, `AverageWidgetProvider.kt`, `ScheduleWidgetProvider.kt`)
8. **Quick actions** — 3D Touch (iOS) / long press (Android)
9. **iOS Spotlight** — Grade search integration
10. **Background sync** — Workmanager, configurable 15 min–24 hours, new grade notifications
11. **7 themes** — Classique, Océan, Forêt, Coucher de soleil, Lavande, Rose, Minuit; auto-switch by time
12. **Demo mode** — `demo@bahut.app` / `demo2024` for Play Store review

---

## Theming

- `core/theme/app_themes.dart` — 7 theme definitions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/menur4) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
