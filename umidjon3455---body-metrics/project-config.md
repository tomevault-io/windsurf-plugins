---
trigger: always_on
description: > This file is designed for general AI coding agents. It provides everything needed to understand, navigate, and contribute to this repository.
---

# AGENTS.md — BodyMetrics Project

> This file is designed for general AI coding agents. It provides everything needed to understand, navigate, and contribute to this repository.

---

## Mandatory Agent Rule

Before making any Flutter/Dart code, test, architecture, UI, localization, DI, routing, or cache/API decision in this repository, **read and follow** [`flutter_rules.md`](flutter_rules.md).

- Treat `flutter_rules.md` as the primary implementation convention guide for this codebase.
- If a generated suggestion conflicts with repo conventions, prefer `flutter_rules.md`.
- Re-check `flutter_rules.md` before major refactors and before writing new feature code.

---

## 1) Project Goal

Health metrics tracking app (Flutter):
- User onboards → picks avatar, enters name/gender/height
- Enters weight → BMI automatically calculated and stored
- Home screen shows historical measurements as cards and charts

---

## 2) Architecture

**Stack:** Flutter + BLoC/Cubit + Clean Architecture + sqflite + Dio (REST API) + auto_route + injectable/get_it

```
lib/
├── core/          # Shared infrastructure (router, theme, widgets, extensions, constants)
├── data/          # SQLite cache + REST API (Dio client, services, interceptors)
├── domain/        # Global entities, use cases, repository contracts
├── feature/       # Feature-based modules (each has presentation + optional domain)
│   ├── splash/
│   ├── onboard/
│   ├── avatar_picker/
│   ├── user_general_info/
│   ├── gender/
│   ├── height/
│   ├── weight_picker/
│   └── home/
└── injection/     # GetIt + Injectable DI setup (locator.dart, locator.config.dart)
```

**Layers (per feature):**
- `presentation/` → View + Cubit + State
- `domain/` → UseCase + Repository (abstract)
- `data/` (in `lib/data/`) → Cache implementations (sqflite) + API services (Dio)

---

## 3) Navigation Flow

Routes defined in `lib/core/router/app_router.dart` (auto_route):

```
SplashView → OnboardView → AvatarPickerView → UserGeneralInfoView
          → GenderView → HeightView → WeightView → HomeView
```

Splash reads `AppModel.page` (enum `Pages`) and `AppModel.isCompleteOnboard` to decide which screen to show.

`Pages` enum values: `onboardPage, avatarPage, userGeneralInfo, genderPage, heightPage, weightPage, homePage`

---

## 4) Critical File Map

### App startup
| File | Role |
|------|------|
| `lib/main.dart` | Entry: `InitApp.init()` → `runApp` |
| `lib/core/init/app_init.dart` | WidgetsFlutterBinding + DB + DI init |
| `lib/feature/splash/presentation/splash.dart` | Reads AppModel → routes user |
| `lib/feature/splash/presentation/splash_model.dart` | Page switch logic |
| `lib/core/router/app_router.dart` | Route definitions |
| `lib/core/router/app_router.gr.dart` | Generated routes (do not edit) |

### State & user session
| File | Role |
|------|------|
| `lib/core/util/models/app_model/app_model.dart` | App-wide state: lastPage, activeUser, isCompleteOnboard |
| `lib/core/util/models/user/user.dart` | User entity (id, name, surname, avatar, gender, height, birthOfDate) |
| `lib/core/util/constants/app_util.dart` | Static: `currentUserId`, `lastPage`, `databaseVersion` |
| `lib/domain/use_case/save_app_use_case.dart` | Persist AppModel to AppCache |
| `lib/domain/use_case/set_id_use_case.dart` | Set AppUtil.currentUserId |

### BMI / metrics calculation & persistence
| File | Role |
|------|------|
| `lib/feature/weight_picker/presentation/cubit/weight_picker_cubit.dart` | Main orchestrator |
| `lib/feature/weight_picker/domain/use_case/calculate_bmi_use_case.dart` | BMI formula: `weight / (height_m²)` |
| `lib/feature/weight_picker/domain/use_case/save_weight_use_case.dart` | Calls SaveWeightRepository |
| `lib/feature/weight_picker/domain/repository/save_weight_repository.dart` | Abstract contract |

### Metric model & cache
| File | Role |
|------|------|
| `lib/core/util/models/user_metrics/user_metric.dart` | UserMetric model (see schema below) |
| `lib/core/util/models/user_metrics/user_metric.g.dart` | Generated JSON serialization |
| `lib/core/util/models/user_metrics/user_metrics.dart` | `UserMetrics` wrapper (List<UserMetric>) |
| `lib/data/cache/user_metrics_cache/user_metrics_cache.dart` | SQLite CRUD |
| `lib/data/cache/user_metrics_cache/user_metrics_columns.dart` | Column name constants |

### Home screen
| File | Role |
|------|------|
| `lib/feature/home/presentation/home.dart` | Main home widget, zoom drawer |
| `lib/feature/home/presentation/widgets/data_list.dart` | Cards list |
| `lib/core/widgets/charts/line_charts.dart` | fl_chart line chart |
| `lib/feature/home/presentation/cubit/user_metric_cubit/user_metric_cubit.dart` | Loads metric history |
| `lib/feature/home/presentation/cubit/home_card_cubit/home_card_cubit.dart` | Toggle list/chart |
| `lib/feature/home/presentation/cubit/user_cubit/user_cubit.dart` | Loads user profile |

### Profile onboarding
| File | Role |
|------|------|
| `lib/feature/avatar_picker/presentation/` | Avatar grid (pr1-pr6) |
| `lib/feature/user_general_info/` | Name, surname, birthdate form |
| `lib/feature/gender/` | Male/female selection |
| `lib/feature/height/` | Ruler-based height picker (65–252 cm) |

### API layer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [umidjon3455/_body-metrics_](https://github.com/umidjon3455/_body-metrics_) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
