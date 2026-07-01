---
trigger: always_on
description: Taurent is a pnpm/Rust monorepo for Tauri-based qBittorrent clients. Keep future work aligned with the existing codemaps and strict frontend/native boundaries.
---

# Taurent — Agent Guide

Taurent is a pnpm/Rust monorepo for Tauri-based qBittorrent clients. Keep future work aligned with the existing codemaps and strict frontend/native boundaries.

## Required read order

1. Read `codemap.md` in the repo root before any task.
2. Read the narrow codemap for the area you will touch:
   - Apps: `apps/codemap.md`, then `apps/desktop/codemap.md` or `apps/mobile/codemap.md`.
   - Shared TS packages: `packages/codemap.md`, then the relevant `packages/*/codemap.md`.
   - Rust/native work: `crates/codemap.md`, then `crates/qb-core/codemap.md`, `crates/qb-tauri/codemap.md`, or the relevant `apps/*/src-tauri/codemap.md`.
3. Then read this file for repo-wide rules and command reminders.

## Toolchain and workspace facts

- Node floor: `>=24.0.0`; CI uses Node `24`.
- Package manager: pnpm `>=11.0.0`; root `packageManager` is `pnpm@11.0.0`.
- JS workspace packages live in `apps/*` and `packages/*`.
- Rust workspace members are `crates/qb-core`, `crates/qb-tauri`, `apps/desktop/src-tauri`, and `apps/mobile/src-tauri`.
- The stack is React 19, Vite 8, Tailwind CSS 4, Tauri 2, Vitest, Playwright, and WebdriverIO for native desktop E2E.

## Package names for `pnpm --filter`

- Desktop app: `taurent`
- Mobile app: `taurent-mobile`
- Shared packages: `@taurent/shared`, `@taurent/bridge`, `@taurent/web-core`, `@taurent/web-ui`

## Commands agents are likely to guess wrong

- Prefer `pnpm desktop:dev`. `pnpm desktop` only starts plain Vite in a browser and skips the native Tauri runtime.
- `pnpm desktop:build` runs the desktop frontend build (`pnpm --filter taurent build`), not a native Tauri bundle. For a native desktop bundle use `pnpm --filter taurent tauri:build`.
- Mobile dev commands are long-lived watch processes: `pnpm mobile:dev`, `pnpm mobile:dev:ios`, `pnpm mobile:dev:android`. Use bounded timeouts when verifying.
- `pnpm mobile:smoke` is `tauri dev` for the mobile app, so it is also long-lived.
- Root `pnpm test:unit` covers `@taurent/shared`, `@taurent/bridge`, `@taurent/web-core`, and `taurent`; it does not include `taurent-mobile`.
- Root `pnpm desktop:ci` includes lint, typecheck, both app frontend builds, unit tests, and desktop renderer E2E. It still does not run native Tauri E2E.

## Focused verification

- Whole JS workspace: `pnpm lint`, `pnpm typecheck`, `pnpm test:unit`
- Local CI smoke: `pnpm ci:local`
- Full local CI path: `pnpm ci:local:full`
- Single package/app: `pnpm --filter <name> lint`, `pnpm --filter <name> typecheck`, `pnpm --filter <name> build`
- Desktop tests:
  - `pnpm desktop:test` -> Vitest
  - `pnpm desktop:test:browser` -> Vitest browser mode
  - `pnpm desktop:renderer:e2e` -> Playwright against mocked desktop renderer runtime
  - `pnpm desktop:tauri:e2e` -> native Tauri E2E runner
- Mobile tests:
  - `pnpm mobile:test` -> Vitest
  - `pnpm mobile:renderer:e2e` -> Playwright renderer E2E
- If you touch Rust, mirror CI with:
  - `cargo fmt --all --check`
  - `cargo check --workspace --locked -p qb-core`
  - `cargo check --workspace --locked --features desktop -p qb-tauri`
  - `cargo clippy --workspace --all-targets --locked -p qb-core`
  - `cargo clippy --workspace --all-targets --locked --features desktop -p qb-tauri`

## Architecture that changes where code belongs

- Layering is strict: `apps/*` -> `@taurent/web-core` + `@taurent/web-ui` -> `@taurent/bridge` -> Rust crates.
- Default new shared logic to `packages/shared`, `packages/web-core`, or `packages/web-ui`; keep `apps/*` thin and platform-specific.
- `packages/shared` owns canonical domain types, schemas, utilities, theme tokens, icon primitives, and small stores.
- `packages/web-core` owns session lifecycle, QueryClient factory/scoping, shared hooks, screen controllers, feature capability probing, and invalidation.
- `packages/web-ui` owns reusable UI primitives, domain components, dialogs, layout components, and screen bodies. Keep it presentational.
- `packages/bridge` owns frontend/native contracts, transport, adapters, logging, and the Tauri API dependency point.
- Rust owns qBittorrent HTTP/auth/session behavior, DTO validation/normalization, synchronization, persistence, filesystem/network/native-platform behavior, and Tauri command implementations when that logic is not purely UI.
- TypeScript/React owns rendering, visual state, route composition, interactions, and presentation view models.

## Frontend boundaries

- `packages/shared`, `packages/web-core`, and `packages/web-ui` must never import `@tauri-apps/*`.
- Route native work through `@taurent/bridge` contracts/adapters or app-specific platform glue.
- `packages/web-core` should stay headless: no UI rendering and no concrete platform imports.
- `packages/web-ui` should not own data fetching, routing, session lifecycle, or platform behavior.
- Apps create a single React Query client at the app shell level; do not introduce extra `QueryClient` instances.
- Both apps initialize `setupTauriLogging()` before importing/rendering `App`; preserve that bootstrap order.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [racos-dev/taurent](https://github.com/racos-dev/taurent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
