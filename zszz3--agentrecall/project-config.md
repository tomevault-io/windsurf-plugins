---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Also read `AGENTS.md` — it holds the binding rules for release notes, dual-app changes, merge/versioning, and safe test practices. This file covers commands and architecture.

## Commands

```bash
npm run setup                # setup:v1 + setup:v2 (do this instead of npm install — see below)
npm run dev:v1               # or dev:v2
npm test                     # test:repo + test:v1 + test:v2
npm run typecheck            # tsc --noEmit in both apps
npm run release-note:check   # MUST pass before opening an MR
npm run package:smoke:all    # build + tarball-install smoke for both apps, in parallel
```

Per-app work is faster scoped:

```bash
npm --prefix apps/main-2.0 test        # vitest run && node --test scripts/*.test.mjs
cd apps/main-2.0 && npx vitest run src/core/postgres/schema.test.ts   # single file
cd apps/main-2.0 && npx vitest run -t "reconciles the OpenViking"     # single test by name
cd apps/main-1.0 && npx vitest         # watch mode
npm run test:workflow-transaction    # V2 workflow-v2 / workspace-transaction gate
npm run test:workflow-performance    # V2 incremental-IPC regression gate
```

Notes:

- **`npm install` alone is not enough.** `scripts/setup-app.mjs` runs `npm ci` with `AGENT_RECALL_SKIP_STATUSLINE_INSTALL=1` (so postinstall does not write into your real `~/.claude`), restores the embedded-PostgreSQL native symlinks that npm packing drops (V2 only), and validates the Electron binary. On Windows, the first `npm run setup:v2` needs an Administrator terminal for those symlinks.
- **There is no ESLint/Prettier/Biome/editorconfig.** `tsc --noEmit` is the only static gate; match surrounding style by hand.
- CI (`.github/workflows/quality-check.yml`) runs on ubuntu/macos/windows: `setup` → `release-notes check-range` → full `npm test` (Linux only) or `test:scripts` (mac/Windows) → `package:smoke:all`. The full suite is **not green on Windows** (POSIX paths, symlinks, fake-CLI spawning) — that is why mac/Windows only run the script tests.

## Two applications, one repo

`apps/main-1.0` (`agent-recall`, stable) and `apps/main-2.0` (`agent-recall-v2`, preview) are independently installed, versioned, and released. The root has **no npm workspaces**; each app has its own lockfile and `node_modules`.

They share concepts but not code. The critical difference: **V1 stores sessions in SQLite with a fully synchronous store API; V2 uses embedded PostgreSQL with a fully async store API.** Per `AGENTS.md`, any session-related change must be inspected in both apps and implemented in both when it applies — but never by copy-paste, because the data layer diverges.

Release channels also diverge: V1 tags `vX.Y.Z` and owns the repo "Latest" marker (V1's updater hardcodes `releases/latest/download/update.json`); V2 tags `v2-X.Y.Z` plus a force-moved `v2-latest` tag that mirrors the stable install and update assets, giving V2 fixed package and manifest URLs without stealing the Latest marker.

## Shared architecture (both apps)

Electron 42 + React 19 + TypeScript 5.7, pure ESM, built by `electron-vite`. Node ≥ 22.13.

- **No path aliases in either app.** Every import is relative (`../../core/...`). Don't introduce `@/`-style aliases.
- **`src/core/`** — Electron-free domain logic: session loaders, the store, indexing, migration, SSH/WSL sync, Supabase sync, skills, quota, provider config. This is where most real logic lives.
- **`src/main/`** — Electron wiring: `index.ts` (a very large god-file: windows, tray, global shortcut, IPC registration, startup sequencing), `main/ipc/` (contract registrars), `main/services/` (injectable, unit-testable service classes).
- **`src/preload/index.ts`** — builds one flat `api` object from per-domain factories, then `contextBridge.exposeInMainWorld("sessionSearch", api)` and `export type SessionSearchApi = typeof api`. That exported type is how the renderer gets end-to-end typing; there is no separate API type declaration to keep in sync.
- **`src/shared/ipc/`** — zod-validated channel contracts. `defineIpcRequest(channel, argsTuple)` + `registerIpcHandler(ipc, contract, handler)` (which returns a disposer). Newer domains use this; older channels are raw `ipcMain.handle("domain:verb", …)` in `main/index.ts` with hand-written preload wrappers. Prefer the contract path for new work.
- **Two renderer windows**: `index.html` (main) and `quick-search.html` (global-shortcut palette), both declared as rollup inputs in `electron.vite.config.ts`.
- **DB path discovery for out-of-process tools**: because Electron's `userData` differs between dev and packaged builds, the app writes a pointer file (`~/.agent-recall/db-path` for V1, `~/.agent-recall-v2/database-url` for V2). The standalone MCP servers in `bin/` deliberately re-implement the resolution logic rather than importing from `src/` — they must run without a build step.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zszz3/AgentRecall](https://github.com/zszz3/AgentRecall) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
