---
trigger: always_on
description: This document is the source of truth for autonomous agents working in the Noah monorepo.
---

# AGENTS.md

## Purpose

This document is the source of truth for autonomous agents working in the Noah monorepo.
It is intentionally operational: where code lives, how the runtime behaves, and how to make safe changes that pass CI.

## Project summary

- Noah is a mobile Bitcoin wallet for Ark (L2).
- Monorepo contains:
  - `client/`: React Native + Expo mobile app.
  - `server/`: Rust Axum backend.
  - `scripts/`: local regtest/dev stack tooling.
  - `fly/`: Fly.io deployment configs.
  - `docs/`: deep-dive docs for push/widgets/notification coordination.

## Monorepo map

- Root
  - `justfile`: primary dev command surface.
  - `flake.nix`: Nix development environments.
  - `Cargo.toml`: workspace root (member: `server`).
  - `package.json`: Bun workspace root (workspace: `client`).
  - `.github/workflows/`: CI/CD pipelines.
- Client
  - `client/App.tsx`: app root (Sentry wrapping, providers).
  - `client/src/Navigators.tsx`: navigation stacks/tabs + onboarding gate.
  - `client/src/AppServices.tsx`: startup side effects (sync, push, backup, server registration).
  - `client/src/lib/`: APIs, wallet wrappers, backup/sync/tasks/logging.
  - `client/src/hooks/`: feature hooks used by UI.
  - `client/src/store/`: Zustand persisted stores.
  - `client/src/types/serverTypes.ts`: generated from server via `ts-rs`.
  - `client/nitromodules/noah-tools/`: custom Nitro module (native bridge).
- Server
  - `server/src/main.rs`: app bootstrap, dependencies, routers, middleware, cron.
  - `server/src/routes/public_api_v0.rs`: public + semi-public API handlers.
  - `server/src/routes/gated_api_v0.rs`: authenticated/gated handlers.
  - `server/src/routes/app_middleware.rs`: auth/user/email middleware.
  - `server/src/db/`: database repository layer.
  - `server/src/cache/`: Redis-backed stores (k1, invoice, email verification, maintenance).
  - `server/src/types.rs`: shared API payloads and enums exported to TS.
  - `server/src/tests/`: integration-style endpoint/repository tests.
  - `server/migrations/`: SQL migrations.

## Tech stack

### Client

- React Native + Expo (bare-style native projects in `ios/` and `android/`).
- Runtime/package manager: Bun.
- TS strict mode.
- State: Zustand with MMKV persistence.
- Data fetching: TanStack Query.
- Styling: Uniwind/Nativewind + `global.css` theme variables.
- Native wallet + crypto: `react-native-nitro-ark`.
- Custom native bridge: `noah-tools` Nitro module.
- Push: Expo Notifications + UnifiedPush fallback (non-GMS Android).

### Server

- Rust (edition 2024) + Axum.
- DB: Postgres (sqlx).
- Cache/state: Redis/Dragonfly (deadpool-redis).
- Push: Expo push and UnifiedPush endpoint POST.
- Jobs: `tokio-cron-scheduler`.
- Storage/Email: AWS S3 + SES.
- Errors: `anyhow` + typed `ApiError` for HTTP responses.

## Runbook

### Setup

- Install deps: `just install` (or `bun install`).
- Enter Nix shell (recommended): `direnv allow` or `nix develop`.

### Mobile app execution policy for autonomous agents

- Do not start Android/iOS apps locally as part of autonomous workflow.
- Do not run simulator/emulator commands like `just android`, `just ios`, or variant-specific equivalents.
- Rely on GitHub Actions client pipelines for platform builds (Android and iOS).

### Server run commands

- Run server locally with live rebuild loop: `just server` (uses `bacon`).
- Build server: `just server-build`.
- Test server: `just server-test` or `just test`.

### Full local regtest stack

- Bring up infra: `just up`.
- Full bootstrap: `just setup-everything`.
- Tear down: `just down`.
- Helpful wrappers: `just bcli`, `just bark`, `just aspd`, `just lncli`, `just cln`.

### Quality checks

- Client checks: `just check` (runs lint + typecheck under client).
- Server checks: `just server-check` and `cargo fmt --check`.
- Combined: `just check-all`.

## CI reality (must match locally)

- Client PR CI (`.github/workflows/client.yml`):
  - `bun client lint`
  - `bun client typecheck`
  - Android signet build
- Server PR CI (`.github/workflows/server.yml`):
  - `cargo fmt --check`
  - `cargo test` with Postgres + Dragonfly services
  - `cargo build --release --bin noah-cli`
- Husky pre-commit runs:
  - `bun client check`
  - `cargo fmt --check`
- After opening a PR, monitor CI status checks:
  - `gh pr checks --watch`
  - or `gh pr view --json statusCheckRollup`
- After PR is merged, sync local `master` before continuing work:
  - `git checkout master`
  - `git pull --ff-only origin master`

## Client architecture details

### App boot sequence

- `client/index.ts` imports `~/lib/pushNotifications` for task registration before root component registration.
- `client/App.tsx` sets providers (QueryClient, SafeArea, GestureHandler, AlertProvider), configures Sentry in non-debug/non-regtest.
- `client/src/Navigators.tsx`:
  - Determines onboarding vs main app based on wallet state.
  - Handles push-permission gate screen.
  - Initializes services via `<AppServices />` after wallet is loaded.

### State model

- `walletStore`:
  - onboarding/wallet-loaded flags, biometrics/debug toggles.
  - background job coordination flags (`isBackgroundJobRunning`, stale cleanup logic).
- `serverStore`:
  - registration status, lightning address, backup enabled, email verified.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [smolcars/noah](https://github.com/smolcars/noah) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
