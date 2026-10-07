---
trigger: always_on
description: > Primary reference for AI coding agents. Read before touching any file.
---

# locationjoystick — Agent Reference

> Primary reference for AI coding agents. Read before touching any file.

---

## Project

Android-only mock GPS app. Background operation, minimal battery.

| Field | Value |
|---|---|
| Package | `com.locationjoystick.app` |
| Language | Kotlin |
| UI | Jetpack Compose |
| Min SDK | API 28 |
| Distribution | GitHub Releases APK + Play Store (AAB) |
| Storage | Room + DataStore |
| Backend | None |
| Open source | Yes |

Constraints:

- Offline-first
- No accounts
- All data on-device in Room + DataStore

---

## Documentation Maintenance Policy

Work is NOT complete until affected docs are updated. These files must stay in sync with the code:

| File | Update when |
|------|-------------|
| `AGENTS.md` (this file) — Feature Specifications table | Adding or removing a feature |
| `AGENTS.md` — Key Services table | Adding, removing, or renaming a service or singleton |
| `docs/architecture.md` — module table | Adding or removing a Gradle module (also update `docs/architecture.md` for architecture pattern changes) |
| `docs/domain-models.md` | Any change to `core/model/` data classes or enums |
| `docs/features/<feature>.md` | Behaviour change in the corresponding feature |
| `docs/features/export-import.md` | Any change to `ExportData` fields or import/export scope |
| `README.md` — feature table | Adding or removing a user-visible feature |
| `docs/wiki/<feature>.html` | Adding or changing any user-visible feature |
| `docs/wiki/changelog.html` + `docs/wiki/changelog/<version>.json` (@docs/features/whats-new.md) | Any release with user-visible changes — the JSON is now the sole authored source for both: it feeds the in-app What's New popup directly, and the HTML is generated from it via `make wiki-changelog` |

Rules:
- New feature → create `docs/features/<feature>.md` AND add row to AGENTS.md's Feature Specifications table AND README.md's feature table.
- New Gradle module → add row to `docs/architecture.md`'s module table (the single source of truth — README.md links to it, doesn't duplicate it).
- New domain model or field → update `docs/domain-models.md`.
- Deleted feature/module → remove from all tables above.
- Doc changes go in the same commit as the code change, not a follow-up.
- New user-visible feature → create `docs/wiki/<feature>.html` AND add it to `NAV_ITEMS` in `docs/wiki/wiki-init.js`. Follow `docs/wiki/CONTRIBUTING.md` for page structure, nav ordering, and writing style. Wiki pages are for **app users, not developers** — no code symbols, class names, Android internals, or library names.
- Wiki prose must pass the audience test in `docs/wiki/CONTRIBUTING.md`: could a non-technical user understand every sentence? If not, rewrite.

---

## Pre-Commit Validation Policy

Work is NOT complete until lint and test passes.

```bash
make format
make lint
make test
```

To verify the AAB builds locally (manual Play Store upload — no automated CI deployment):

```bash
make bundle
```

Rules:
- Fix every lint error before declaring done. Warnings acceptable; errors not.
- Run after every set of edits, not just end of session.
- If check fails, fix root cause. Don't suppress unless genuine false positive + inline comment explaining why.
- Never suppress `Errors` category rules. Never batch-suppress with `@file:Suppress`.
- Never add co-authoring or "Claude-Sessions" to the commit

---

## Architecture

→ See @docs/architecture.md

---

## Constants

→ See @docs/constants.md

---

## Feature Specifications

→ See @docs/features/

| Feature | Doc |
|---------|-----|
| Mock Location Engine + GPS Realism | @docs/features/mock-location.md |
| Foreground Service | @docs/features/foreground-service.md |
| Floating Joystick | @docs/features/joystick.md |
| Map (MapLibre) | @docs/features/map.md |
| Route System | @docs/features/routes.md |
| Favorite Locations | @docs/features/favorites.md |
| Speed Profiles | @docs/features/speed-profiles.md |
| Floating Widget | @docs/features/widget.md |
| Click-to-Move / Teleport | @docs/features/click-to-move.md |
| Roaming Mode | @docs/features/roaming.md |
| Export / Import | @docs/features/export-import.md |
| QR Share / Transfer | @docs/features/qr-transfer.md |
| Deep Links & Location Sharing | @docs/features/deep-link.md |
| Last Remembered Location | @docs/features/last-location.md |
| Onboarding | @docs/features/onboarding.md |
| Group Sync | @docs/features/group-sync.md |
| Tap to Walk | @docs/features/tap-to-walk.md |
| Theme | @docs/features/theme.md |
| Hide Teleport Features | @docs/features/hide-teleport.md |
| What's New Popup | @docs/features/whats-new.md |

---

## Domain Models

→ See @docs/domain-models.md

---

## Key Services

| Service | Module | Type | Purpose |
|---------|--------|------|---------|
| `MockLocationService` | `:core:location` | ForegroundService | Owns `LocationManager` test provider. Exposes `StateFlow<SpoofState>`. Commands: `startSpoofing`, `updatePosition`, `stopSpoofing`. Suspended-phase state held in `AtomicReference<SuspendedPhaseState>`; transitions via `advanceSuspendedPhase()` pure function (testable independently). |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shortcuts/locationjoystick](https://github.com/shortcuts/locationjoystick) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
