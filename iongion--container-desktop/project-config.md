---
trigger: always_on
description: > **`AGENTS.md` is a symlink to this file.** Edit only this file; both names
---

# Project Guide — container-desktop

> **`AGENTS.md` is a symlink to this file.** Edit only this file; both names
> resolve to the same content. Keep it ≤ 200 lines.

## What this is

Cross-platform **Electron desktop app** for managing container engines
(Podman, Docker, and Apple Container — the last is **experimental**, macOS /
Apple-silicon only) — local, remote over SSH, and WSL. One repo, three runtimes:

- **Node / TypeScript** app — Electron main + preload + React renderer
- **Go** SSH/vsock relay — `support/container-desktop-relay/`
- **Python** build tooling — `invoke` tasks (`tasks.py`) + `uv`

## Stack

Electron 42 · React 19 · Vite 8 (rolldown) · TypeScript 6 · Blueprint 6 (UI) ·
Zustand (state) · TanStack Query + Router · @xterm/xterm 6 · bundled monaco · Biome (lint/format).
Node **24.16.0** (`.nvmrc`), **yarn 1.x** (classic). Go 1.25+ (toolchain 1.26.4).
Python ≥ 3.12 via `uv`.

## Layout

- `src/electron-shell/` — `main.ts`, `preload.ts`, `shared.ts`
- `src/web-app/` — React renderer: `App.tsx`, `stores/` (Zustand state),
  `domain/` (TanStack Query client), `screens/`, `components/`, `hooks/`,
  `Native.ts`, `Environment.ts`
- `src/container-client/` — engine API clients/adapters · `src/logger/` ·
  `src/utils/` · `src/env/`
- `src/platform/` — Node `Command` primitives: `node-executor.ts` (facade) over
  `exec/` impl modules (process-utils, api-driver, commander, ssh-transport,
  wsl-relay, proxy-request); `node.ts` = `Platform`/`Path`
- `vite.config.{common,main,preload,renderer}.mjs` · `electron-builder-config.cjs`
  · `support/watch.mjs` (dev launcher) · `tasks.py` / `Makefile`
- **`website-src/`** — Eleventy sources for the public site (container-desktop.com),
  compiled to the **generated `website/`** (never hand-edit `website/`; see Website below).
- **`docs/`** — architecture docs (C4 diagrams) + contributor guides;
  start at `docs/README.md`.
- Path alias **`@/* → src/*`** (e.g. `@/web-app/...`), defined in `tsconfig.json`
  `paths` + explicit `resolve.alias` in the common vite config.

## Commands

Use the project Node first: `nvm use` (24.16.0). Package manager is **yarn**.

- Install: `uv run --locked invoke prepare` or `yarn install --frozen-lockfile`
- **Verify — run all four before claiming done:**
  `yarn check-types` (tsc) · `yarn lint` (Biome; `yarn lint:check` = no-write, used in CI) ·
  `yarn test:run` (Vitest, hermetic) · `yarn build` (main+preload+renderer)
- Dev (hot reload): `yarn dev` · Format: `yarn format`
- Package: `yarn package:linux_x86` (also `mac_arm`/`win_x86`/`linux_arm`);
  full release: `inv release`
- Publish GitHub release assets locally only:
  `uv run --locked invoke publish-release --run-id <actions-run-id>` dry-run,
  then add `--perform`. The Microsoft Store wrapper is optional and can be
  copied into `release/container-desktop-installer.exe` when available.
  `CDPipeline.yml` can also publish after all production targets build; use its
  `replace-release` input to delete/recreate the same version cleanly.
- Relay: `cd support/container-desktop-relay && ./relay-build.sh`; scan `govulncheck ./...`
- Python tooling: `make check` (ruff), `make prepare` (`uv sync --locked --dev --no-install-project`)
- Linux system deps (one-shot): `bash support/provision-deps.sh`

## Development workflow — TDD + live app, NOT static-checks-at-the-end (non-negotiable)

How you build here, **per change** — not an end-of-task afterthought:

- **Test-first (TDD) for logic.** Failing test → watch it fail for the right reason → minimal
  code to pass. Covers pure/near-pure units: hook helpers, `normalizers/`, `comparators`,
  grouping/flatten, reducers, stores. No production logic without a failing test first — tests
  added after prove nothing. Layout/DOM glue jsdom can't run (virtualizer, ResizeObserver) is
  exempt: verify it live, don't fake it.
- **Verify every UI change in the running app, as you go.** Keep `CONTAINER_DESKTOP_MOCK=1 yarn
  dev` hot-reloading and drive `support/cdp.mjs` (screenshot + `EVAL=` asserts) after each
  change — never batch to the end. The renderer is the source of truth.
- **Static checks close out, they aren't the loop.** `check-types`/`lint`/`test:run`/`build`
  never render the UI; run all four (Commands) only to finish, after the app confirms behavior.

## Build / runtime model — READ BEFORE TOUCHING THE BUILD

- **Source is ESM/TypeScript, but main & preload are bundled to CommonJS (`.cjs`).**
  Electron's API is only reachable via the CJS `require` hook here; ESM output
  makes `import { app } from "electron"` fail to link. **Do not switch main/preload
  to ESM output.** The **renderer stays ESM**.
- `__dirname` is native in the CJS main/preload (the common config only maps it to
  `import.meta.dirname` for ESM output).
- **Preload** builds to `build/<version>/preload.cjs`; the renderer blocks on
  `window.Preloaded`, exposed via `contextBridge` in `preload.ts`
  (see `Native.ts:waitForPreload`).
- **Production requires `ENVIRONMENT=production`** (e.g.
  `cross-env ENVIRONMENT=production yarn build`): emits the single-file
  `build/<version>/{main.cjs,preload.cjs,renderer.mjs}` layout and loads the
  packaged renderer over `file://`. `ssh2` is bundled by rolldown `ssr.noExternal`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iongion/container-desktop](https://github.com/iongion/container-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
