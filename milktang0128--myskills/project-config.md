---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**MySkills** — an Electron desktop app (macOS-only for MVP) that aggregates AI agent skills across Claude Code, Codex, and a user-maintained shared pool into a single searchable, scenario-organized hub. The authoritative product spec is [SPEC.md](SPEC.md) (Chinese) — any implementation that diverges from it should update the spec first.

Current state: **MVP-A is built** (read-only inventory + scenarios + Discover via skills.sh + optional BYOK AI). MVP-B (writes: sync, enable/disable) is partially in place — `electron/sync/` and the `sync:*` IPC channels exist, but the renderer-visible UI is gated behind plan-confirm flows. See [README.md](README.md) for the user-facing feature list.

## Commands

```bash
npm run dev          # Runs Next dev (port 4477) + Electron concurrently, with wait-on
npm run dev:next     # Next.js renderer only
npm run dev:electron # Electron main process only (tsx, waits for tcp:4477)
npm run build        # next build → out/, then tsc electron → dist-electron/
npm run package      # build, then electron-builder --mac (DMG)
npm run rebuild      # electron-rebuild for better-sqlite3 (run after npm install fails native build)
```

No test runner or linter is wired up yet — don't claim "tests pass" without first wiring one in.

## Architecture

### Two TypeScript projects, one repo

The renderer and main process are compiled by **separate `tsconfig.json` files**, and the root `tsconfig.json` explicitly **excludes `electron/`**. Don't import renderer code from `electron/` or vice versa — only `shared/` is allowed across the boundary.

- `tsconfig.json` (root) — Next.js renderer. `module: esnext`, `jsx: preserve`, `noEmit`. Path aliases: `@/*` → `src/*`, `@shared/*` → `shared/*`.
- [electron/tsconfig.json](electron/tsconfig.json) — Main process. `module: CommonJS`, emits to `dist-electron/`. `rootDir: ..` so `shared/` compiles in too. Excludes `dev.ts` (run via tsx in dev).

### Static-export Next.js loaded over `file://`

[next.config.mjs](next.config.mjs) uses `output: 'export'` + `distDir: 'out'` + `trailingSlash: true`. In production Electron loads the static bundle via `file://`, so `assetPrefix: './'` is set for production only. **Consequences:**
- No Next.js server features (no API routes, no SSR, no middleware, no Server Actions).
- All dynamic data flows through Electron IPC.
- Any link/asset paths must be relative-safe.

### IPC and security model (SPEC §5.3)

- All filesystem and SQLite work lives in the **main process**. The renderer reads cached data via IPC only.
- IPC goes through `contextBridge` + `ipcMain.handle()`; the renderer runs with `nodeIntegration: false` and only sees a whitelisted API surface.
- `shared/types.ts` is the contract between processes — keep it **dependency-free** (it's imported by both sides).
- **Adding a new IPC channel requires an Electron restart, not just a renderer reload.** The preload is esbuild-bundled at `dev:electron` startup ([electron/build-preload.mjs](electron/build-preload.mjs)) to allow `sandbox: true`, which freezes the `ALL_INVOKE_CHANNELS` whitelist into the bundle. The renderer will see `Channel "x:y" is not allowed` until you kill + restart Electron. Workflow: add channel to [shared/ipc-channels.ts](shared/ipc-channels.ts) → restart `npm run dev:electron` (Next stays up).

### Main-process layout

`electron/` is grouped by concern, not by feature. Routes are wired in [electron/ipc/index.ts](electron/ipc/index.ts) via `registerAllHandlers()`, which is called once from `app.whenReady()` in [electron/main.ts](electron/main.ts).

- `electron/paths.ts` — **the single point** where `app.getPath('userData')` enters the system. `initPaths()` runs before `initDb()`; everything downstream (DB file, backups, staging dirs) reads from this module rather than calling `app.getPath` directly. Keep that invariant.
- `electron/secrets/` — same DI pattern: `setSecretStore(electronSafeStorage)` installs the Keychain-backed implementation; the rest of the codebase imports the interface from `safe-storage.ts`. Tests / non-Electron callers swap the store.
- `electron/db/` — schema + migrations + `index.ts` (initDb). `better-sqlite3` is opened once and reused.
- `electron/scanner/` — walks platform skill dirs, parses `SKILL.md` frontmatter via `gray-matter`, classifies platforms. `maybeAutoScan` runs on launch.
- `electron/ipc/` — one file per domain (`platforms`, `skills`, `scenarios`, `scan`, `coverage`, `sync`, `catalog`, `llm`, `ai`, `settings`). `dispatcher.ts` enforces sender validation: `setAllowedSender(mainWindow.webContents)` is called before handlers register, and every `ipcMain.handle` rejects events whose `sender` is not the pinned main window.
- `electron/sync/` — plan/execute/backup primitives for the symlink and copy modes.
- `electron/catalog/` — skills.sh client + GitHub raw enrichment + install planner (the Discover view's backend).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Milktang0128/myskills](https://github.com/Milktang0128/myskills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
