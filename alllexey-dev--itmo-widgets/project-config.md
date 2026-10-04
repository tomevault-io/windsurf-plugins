---
trigger: always_on
description: This file is the working agreement for anyone changing the ITMO.Widgets
---

# ITMO.Widgets agent guide

This file is the working agreement for anyone changing the ITMO.Widgets
ecosystem. It states the rules that are not derivable from the code. Detailed
references live in `docs/`; this file only points at them. A nested `AGENTS.md`
in a sibling repository adds repository-specific rules and never contradicts this one.

## Repositories

| Component | Local path | Remote | Owns |
|---|---|---|---|
| Android app | `/Users/alllexey/proj/ITMO.Widgets` | `alllexey-dev/ITMO.Widgets` | UI, local caches, widgets, debug fixtures, this guide |
| Legacy app | tag `v2.0.1` in the same repository | same remote | Read-only reference for v2.0.1 parity (`git show v2.0.1:<path>`); no separate checkout |
| Core | `/Users/alllexey/proj/itmo-widgets-core` | `alllexey-dev/itmo-widgets-core` | Typed Retrofit contract and client for Backend |
| Backend | `/Users/alllexey/proj/itmo-widgets-backend` | `alllexey123/itmo-widgets-backend` | Users, friendships, privacy, sport queues, FCM delivery |
| MyItmoApi | `/Users/alllexey/proj/MyItmoApi` | `alllexey123/MyItmoApi` | Typed Java client for official MyITMO and BARS |

They are independent Git repositories: inspect status, diff, tests and commits
separately, and report a commit hash per repository.

## Where things are documented

| Topic | Document |
|---|---|
| Release sequence, version compatibility, deferred features | `docs/product/releases.md` |
| Delivery order of the social and study features | `docs/product/roadmap.md` |
| Android package structure, layers, enforced rules | `docs/architecture.md` |
| Visual language, spacing, states, verification matrix | `docs/design.md` |
| User-facing settings and their defaults | `docs/settings.md` |
| Per-feature behaviour and invariants | `docs/features/*.md` |
| Decisions that were argued once and must not be re-litigated | `docs/decisions/*.md` |
| Backend contracts, database and deployment | `../itmo-widgets-backend/docs/` |
| Core wire conventions | `../itmo-widgets-core/docs/contract.md` |
| Plans and progress trackers for work in flight | `vibe/` (ignored, local only) |

Documents describe the current state. History goes to each repository's
`CHANGELOG.md`, never into a reference document.

## Source-of-truth boundaries

- **MyItmoApi** is the only client for official MyITMO and BARS endpoints. Extend
  it instead of adding a second Retrofit interface anywhere else. Models document
  the observed response shape, units, enum-like values and whether null was
  actually observed; unknown fields stay as commented declarations, not `Any?`.
- **Core** is the typed contract for Backend. Backend changes first, Core mirrors
  them with serialization and MockWebServer tests, Android consumes last.
  Retrofit annotations mirror the controllers exactly.
- **Backend** is authoritative for social data and authorization. It returns
  viewer-scoped capabilities, never another user's raw privacy settings, and does
  not trust the client to enforce access.
- **Android** owns presentation, caches, widgets and local preferences. It never
  enforces privacy on behalf of Backend.

Cross-repository change order: privacy boundary → Backend + tests → Core + tests
→ `publishToMavenLocal` → Android → test against `https://dev.widgets.alllexey.dev`
→ release and deploy only on explicit request.

## Hard rules

- Academic logic takes time from `AcademicTimeProvider` or an injected `Clock`.
  No direct `now()` calls in feature code; the Konsist suite enforces it.
- Android stays on XML, Fragments, ViewBinding, Navigation, Hilt and WorkManager.
  No Compose without an explicit migration decision.
- Dependency direction inside a feature is `ui -> presentation -> domain <- data`;
  features never import each other. See `docs/architecture.md`.
- Everything that reaches Backend is gated on the custom-services opt-in inside
  the repository layer.
- Synthetic data lives in debug fixtures and the demo session (`DemoMode`); it
  never reaches Backend. Every class that takes a network client checks
  `DemoMode` before the call; the Konsist suite enforces it.
- User-visible text lives in string resources and is Russian.
- Every meaningful UI change is verified on an emulator in light and dark theme,
  one dynamic palette, font scale 1.0 and 1.3, with long names and every state.
  Compilation is not visual verification.

## Secrets and data

Never commit or paste `.refresh-token`, `local.properties`, `app-keystore.jks`,
Backend `.env`, database dumps, Firebase keys, tokens or user data. Redact
command output before quoting it. ITMO credentials are entered only on official
ITMO pages; refresh tokens stay on the device. Backend authenticates requests
with the ITMO.ID access token and never stores the user's refresh token.
Never mutate production data for testing.

## Build and verify

Run commands from the repository they belong to. Every affected repository must
build and test before a cross-repository change is called done.

```bash
# Android (JDK 17+): both distribution variants, github (APK) and play (AAB)
./gradlew :app:testGithubDebugUnitTest :app:testPlayDebugUnitTest :app:lintGithubDebug :app:lintPlayDebug \
  :app:assembleGithubDebug :app:assemblePlayDebug

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alllexey-dev/ITMO.Widgets](https://github.com/alllexey-dev/ITMO.Widgets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
