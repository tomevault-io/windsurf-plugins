---
trigger: always_on
description: This file is the working guide for AI agents and human contributors inside this repository. Follow it together with [CONTRIBUTING.md](CONTRIBUTING.md) and [CONTRIBUTING.zh.md](CONTRIBUTING.zh.md).
---

# DeepOrganiser Agent Guide

This file is the working guide for AI agents and human contributors inside this repository. Follow it together with [CONTRIBUTING.md](CONTRIBUTING.md) and [CONTRIBUTING.zh.md](CONTRIBUTING.zh.md).

## Project Core

DeepOrganiser is an Electron + React desktop app and standalone WebUI shell around a bundled Rust backend binary named `deeporganiser-core`.

The important mental model:

- The TypeScript/Electron code owns the app shell, renderer UI, packaging, WebUI static server, Electron/Web bridge, config client, type contracts, migrations from legacy stores, bundled MCP scripts, and local process lifecycle.
- `deeporganiser-core` owns most runtime business APIs: auth, users, providers, conversations, messages, MCP, cron, agents, remote agents, teams, assistants, settings, and realtime events.
- The renderer almost never talks to Electron main directly for business data. It calls the backend through `ipcBridge`, which is now mostly HTTP/WebSocket wrappers over `/api/*` and `/ws`.
- Electron desktop and standalone WebUI share the same renderer bundle, but differ in transport and auth. Desktop injects a backend port through preload. WebUI uses same-origin `/api/*`, `/login`, `/logout`, and `/ws` proxied by `@deeporganiser/web-host`.

When something is not implemented in TypeScript, check whether it is an `deeporganiser-core` API surface before adding duplicate frontend/main-process logic.

## Repository Map

Top-level layout:

- `packages/desktop/`: Electron app, renderer, preload, main process utilities, shared TS contracts.
- `packages/web-host/`: starts or attaches to `deeporganiser-core`, serves `out/renderer`, proxies `/api/*`, `/login`, `/logout`, `/ws`, and `/api/stt/stream`.
- `packages/web-cli/`: CLI helpers for browser opening and WebUI admin password flow.
- `packages/shared-scripts/`: shared release/setup helpers, including bundled `deeporganiser-core` preparation/verification.
- `scripts/`: build, packaging, WebUI launch, migrations, debug, i18n, and release utilities.
- `resources/bundled-deeporganiser-core/<platform>-<arch>/`: expected location for the `deeporganiser-core` binary in dev/package flows.
  This directory is gitignored and usually absent in a fresh worktree.
- `docs/contributing/`: contributor setup and structure rules. Some references in `docs/README.md` point to directories not present in this clone, so prefer source when docs disagree.
- `docs/prds/`: product requirements. Do not reorganize product-owned docs without explicit permission.
- `examples/`: extension examples, including ACP adapters, channels, assistants, skills, themes, and settings tabs.
- `.claude/skills/`: repository-local skills. Use them when their trigger conditions apply.

## Runtime Architecture

Desktop startup:

1. `packages/desktop/src/index.ts` runs first. It configures Chromium/app paths before any `app.getPath('userData')` call, enforces single-instance behavior, fixes GUI PATH, initializes Sentry/logging, starts storage, starts `deeporganiser-core`, creates the main window, and wires Electron lifecycle/tray/deep-link/WebUI behavior.
2. `packages/desktop/src/process/index.ts` initializes platform registration, storage, bridges, and main-process i18n.
3. `@deeporganiser/web-host` `BackendLifecycleManager` resolves `deeporganiser-core` via `packages/desktop/src/process/backend/binaryResolver.ts`, spawns it, waits for health/listening, and stores the backend port on `globalThis.__backendPort`.
4. `packages/desktop/src/preload/main.ts` exposes the Electron bridge and backend startup state to the renderer.
5. `packages/desktop/src/renderer/main.tsx` initializes config/i18n/theme, installs the browser bridge adapter, prefetches detected agents, and mounts the React app.

Standalone WebUI startup:

1. `scripts/webui.ts` optionally runs `bun run package`, resolves renderer assets, resolves `deeporganiser-core`, chooses a data dir and port, then calls `startWebHost`.
2. Default WebUI ports are dev `25809`, multi-instance dev `25810`, production `25808`.
3. Default standalone data dirs are `~/.openscience-web-dev`, `~/.openscience-web-dev-2`, or `~/.openscience-web`; they intentionally do not collide with Electron's default data dir.
4. `packages/web-host/src/static-server.ts` is a static server and reverse proxy only. Do not put business routes there.

Backend spawning:

- `packages/web-host/src/backend-launcher.ts` builds `deeporganiser-core` args: `--port`, `--data-dir`, optional `--parent-pid`, `--log-level`, `--app-version`, `--managed-resources-mode bundled`, `--log-dir`, `--work-dir`, and `--local`.
- It injects `DEEPORGANISER_CACHE_DIR`, `DEEPORGANISER_WORK_DIR`, and `DEEPORGANISER_LOG_DIR` so backend `/api/system/info` matches Electron-managed directories.
- It cleans registered child agent processes through `runtime/agent-process-registry.json` before backend startup.
- It skips Fetch-forbidden ports and validates startup with health checks.

## Local Backend Resources

`deeporganiser-core` is not built from this TypeScript repository during normal app startup. The desktop app expects a prebuilt backend bundle at:

```text

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ResearAI/OpenScience](https://github.com/ResearAI/OpenScience) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
