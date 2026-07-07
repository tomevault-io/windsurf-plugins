---
trigger: always_on
description: This is a clean-room rebuild of `/Users/hillel/Code/admin-portal`. Read this file before changing anything substantial.
---

# Invoice Ninja — Flutter App (rebuild)

This is a clean-room rebuild of `/Users/hillel/Code/admin-portal`. Read this file before changing anything substantial.

## What this app is

A multi-platform Invoice Ninja admin client. Replaces the Redux-based admin-portal with three goals:
1. **Page-by-page data loading** — never `per_page=999999`.
2. **True offline editing** — every change lands in a local mutation outbox and syncs when online.
3. **No Redux** — plain Flutter state management.

Plus two non-negotiables carried from admin-portal:
- App restart restores exactly where the user left off (route, company, filters).
- Multi-company support.

## Quick Index

| When you're doing… | Look at |
|---|---|
| Adding a new entity | § Adding a new entity + `docs/adding-an-entity.md` |
| Adding / editing a settings screen | § Settings screens + `docs/settings-screens.md` |
| Wiring a form field, picker, or Enter-to-save | § Forms |
| Anything money / date / parsing | § Strict rules + § Forms |
| Dialog buttons rendering stacked | § Design system (v2) |
| Sync / outbox / 401-403-404-409-412-422 behavior | § Sync — non-obvious rules |
| Bundled vs per-entity data loading | § Data loading — bundled vs per-entity |
| Architecture, write pipeline, project layout | § Architecture — at a glance + `docs/architecture.md` |
| Changing the Drift schema (forward migration) | `docs/migrations.md` |
| Localization / Transifex import | § Localization |
| Cross-checking against legacy admin-portal / React / API docs | § Reference points |
| macOS entitlement, dev login pre-fill, platform targets | `docs/setup.md` |
| Building a release app / injecting the Sentry DSN | `tools/build_release.sh` + `docs/setup.md` § Release builds with Sentry |
| Probing the demo API for live response shapes | `docs/probing-the-demo-api.md` |
| Server-side filter gaps / required API changes | `BACKEND.md` |
| Running integration tests | `docs/integration-tests.md` |
| Debugging a runtime error or stale outbox row | § Diagnostics log + `docs/diagnostics.md` |
| Desktop window persistence (native runners) | `docs/desktop-window-state.md` |
| Rotating the `is_system` API token (blocked on server) | `docs/token-rotation.md` |
| Checking what's built vs what's left | `FEATURES.md` (kept current — see § Strict rules) |
| Working around an open upstream (Flutter/pub) bug — or undoing one later | `docs/upstream-workarounds.md` |

## Strict rules

Rules that turn into bugs or CI failures if forgotten. Read this block first.

- **No Redux. No bloc. No Riverpod.** `ChangeNotifier` only. Tempted to add one? Talk to the team first.
- **No `per_page=999999`.** Lists fetch one page at a time (50 rows default): the ViewModel calls `repo.ensurePageLoaded(N)` near the scroll edge, the repo writes the page to Drift, the UI reacts via the watch stream. A CI lint grep-fails the build if the literal appears in `lib/`.
- **Money is `Decimal`, never `double`.** Enforced by a CI test (greps entity models).
- **Date-only is the custom `Date` type; `DateTime` is for timestamps only.** Mixing them silently breaks invoice math.
- **Drift is the only thing the UI reads from.** The network writes to Drift; the UI watches Drift. Never read API responses straight into UI state.
- **Schema changes need a forward Drift migration now (post-beta).** The app is shipped — installed databases hold real user data and unsynced outbox edits. Any schema change (table / column / index) must bump `AppDatabase.schemaVersion`, add an `onUpgrade` step, re-dump (`drift_dev schema dump`) + re-generate (`schema generate`), and extend the matrix test. **Never re-squash to v1 or overwrite a shipped `drift_schemas/drift_schema_v*.json`** — a frozen-checksum CI test (`test/data/db/migration_test.dart`) fails the build if you do. Skipping the migration silently wipes the user's local DB (and pending offline edits) via the `isSchemaIntact()` reset backstop. Full workflow: `docs/migrations.md`.
- **Auth user data flows in through `/refresh`, not `GET /users/{id}`.** `_persistAndActivate` upserts each `data[N].user` block into the `users` Drift table on every login/refresh. `GET /users/{id}` is 412-gated (password-required). `auth.refresh()` runs a full `_persistAndActivate` — use it for a fresh session snapshot, not for incidental work. Never call `UsersApi.get` from incidental paths.
- **Every write goes through the outbox.** Repositories never call mutation endpoints directly.
- **Every list query is scoped by `company_id`.** Use `CompanyScopedDao` — direct table access bypassing the DAO fails a lint check.
- **Idempotency keys are stable across retries** — generated when the outbox row is created, reused on every retry.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [invoiceninja/flutter](https://github.com/invoiceninja/flutter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
