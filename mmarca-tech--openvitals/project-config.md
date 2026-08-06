---
trigger: always_on
description: Implementation guide for coding agents working in this repository.
---

# AGENTS.md

Implementation guide for coding agents working in this repository.

Read this before adding a feature, extending a metric screen, or touching health, l10n, or background code.

This app is a 1:1 Flutter port of the Kotlin OpenVitals app, which it REPLACED in place on this same repository (same Play listing, same package `tech.mmarca.openvitals`, same signing key). The Kotlin sources no longer exist in the working tree -- read them from git history: `git show 23c14d0:app/src/main/kotlin/...`. Behaviour parity with the Kotlin source is the default requirement; deviate only with a reason, and write the reason down.

## Source Of Truth

- [docs/README.md](docs/README.md): doc index
- [docs/engineering/architecture.md](docs/engineering/architecture.md): architecture and target direction
- [docs/engineering/feature-playbook.md](docs/engineering/feature-playbook.md): step-by-step guide for adding a feature
- [docs/features/feature-map.md](docs/features/feature-map.md): feature to route/screen mapping
- [docs/engineering/translations.md](docs/engineering/translations.md): ARB, Weblate, and the l10n gate

If code and docs disagree, prefer the docs for new work and refactor toward them incrementally.

Caveat while the port completes: the docs under `docs/engineering/` still carry Kotlin-era mechanics in places (Gradle tasks, Hilt, Compose). Their *principles* are binding; their *Kotlin specifics* are stale — the Flutter equivalents are in this file and in `README.md`.

## Golden Path For A New Metric Feature

1. Define the feature contract: screen state, user actions, derived display fields.
2. Make it period-driven: `Day / Week / Month / Year`, a selected anchor date, previous/next navigation, capped at the current period.
3. Keep the frame reusable, keep the charts specific: reuse `lib/ui/components/metric_detail_scaffold.dart` and `lib/ui/components/period_navigator.dart`; keep metric-specific cards and charts inside the feature directory.
4. Keep repository APIs query-oriented: pass a `DatePeriod` (`lib/core/period/`) or a query object from `lib/domain/query/`, not another ad hoc `loadX(start, end)` overload.
5. Register the feature from the dashboard: dashboard card, route in `lib/navigation/app_routes.dart` + `lib/navigation/app_router.dart`, screen title.
6. Declare what the screen reads: pass `refreshDomains` to `MetricDetailScaffold` (or mix in `RefreshOnSignal` if the screen does not use it). See *Refreshing* below.
7. Update the docs if the pattern evolves.

## Layout Rules

Feature code lives under `lib/features/<feature>/`, split into two subdirectories: `application/` (the view-model, its `freezed` state, and — as features migrate — the pure `build<X>Display` functions) and `presentation/` (screens, cards, charts). Feature sub-domains keep their own subdirectory (`reminders/`, `applehealth/`, `maps/`; settings cards live in `presentation/cards/`). See `lib/features/sleep/` or `lib/features/heart/` for the intended shape. `homewidgets/` is the one flat exception — background-isolate glue with no view-model.

Shared code lives in:

- `lib/ui/components/` — reusable shell components (no feature business logic)
- `lib/ui/charts/`, `lib/ui/theme/` — shared chart and theme primitives
- `lib/core/period/` — period math and window formatting
- `lib/core/presentation/` — repository-free formatters and UI models
- `lib/domain/model/`, `lib/domain/insights/`, `lib/domain/preferences/` — pure models, calculations, preference enums
- `lib/data/repository/contract/` + `impl/` — the repository boundary
- `lib/di/providers.dart`, `lib/state/app_providers.dart` — provider wiring

### State

One view-model per screen — a Riverpod `Notifier` / `AsyncNotifier` subclass named `<X>ViewModel` in `application/<x>_view_model.dart` — with state as a `freezed` class. (MVVM per the Flutter app-architecture guide; the Riverpod notifier IS the view-model, so nothing feature-side carries the Notifier suffix.) A view-model owns loading state, owns the selected range/anchor date, calls use-cases/repositories, and exposes UI-ready state. It must not carry large formatting blocks (that is `lib/core/presentation/`), must not re-implement period math, and must not mirror raw Health Connect record shapes when a cleaner UI model is warranted.

The one exception to `freezed`: when the whole state **is** a single already-immutable value, `Notifier<int?>` / `Notifier<double>` / `Notifier<SomeValueObject>` is the state. A freezed wrapper around it would add a field and no information. Three settings card view-models are like this; everything else is freezed.

**A widget never holds a repository** — not even to read a synchronous constant off one. If a screen needs a permission set for its `HealthConnectGate`, it watches a provider (`mindfulnessWritePermissionsProvider` and friends in `lib/di/data_providers.dart`), not `ref.watch(mindfulnessRepositoryProvider)`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mmarca-tech/OpenVitals](https://github.com/mmarca-tech/OpenVitals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
