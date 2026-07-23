---
trigger: always_on
description: Guidance for Claude Code working in this repo. Project is a starter template for desktop apps that combine **Electron** (shell), **React + Redux Toolkit** (UI), and **Python/Flask** (backend microservice). Three processes run together; the wiring is the interesting part. The renderer, Electron main + preload, and build scripts are all TypeScript.
---

# CLAUDE.md

Guidance for Claude Code working in this repo. Project is a starter template for desktop apps that combine **Electron** (shell), **React + Redux Toolkit** (UI), and **Python/Flask** (backend microservice). Three processes run together; the wiring is the interesting part. The renderer, Electron main + preload, and build scripts are all TypeScript.

## Scope discipline (read this first, every session)

Two load-bearing rules. The user has called both out explicitly. Re-read at the start of every session.

### Rule 1 — minimum diff

**Do not introduce files, splits, refactors, or restructuring beyond what the current task literally asks for.** Specifically:
- Do **not** split `requirements.txt` into `requirements.txt` + `requirements-dev.txt`. Keep one file. Same for any analogous "runtime vs dev" or "best-practice" expansion.
- Do **not** add new files (CONTRIBUTING.md, configs, helper modules, fixtures folders) unless the task literally names them.
- Do **not** rename, reformat, or alphabetize unrelated code while in the area.
- Do **not** run long-lived dev processes (`yarn start`, `electron .`, `vite`) for smoke tests without explicit user OK and a guaranteed kill in the same call. Orphan processes have happened — don't repeat.
- If a side-fix is genuinely required for the requested task to land (e.g., a lint error blocking `yarn verify`), say so explicitly **before** doing it, and only the smallest version.

Memory: `feedback_no_unscoped_changes.md`.

### Rule 2 — packaged app is zero-install for end users

**Developers** cloning this template need Node + Python + `yarn install` + `pip install -r requirements.txt`. That's expected — the README setup section is for them.

**End users** who install the resulting MSI / DMG / DEB must NOT need Python, Node, pip, yarn, npm, virtualenvs, system libraries, or any other runtime. They double-click the installer and the app runs — like any other consumer desktop app.

Practical implications:
- Python runtime deps must be bundled by PyInstaller (`app.spec` `hiddenimports` / `datas`).
- Node runtime deps used by `main.ts` / `preload.ts` go in `dependencies`, not `devDependencies` (devDeps don't ship in the asar).
- Production code paths must not shell out to `pip`, `npm`, `node`, `python`, `git`, `make`, or any binary that may be absent on a clean Windows / macOS / Linux machine.
- No first-run installers, no postinstall download steps, no "install Python first" prompts.
- If a feature genuinely cannot ship without forcing an end-user install, stop and surface the constraint **before** implementing it.

Memory: `feedback_packaged_app_zero_runtime_deps.md`.

## Quick map

| Layer | Entry | Notes |
|---|---|---|
| Electron main | [main.ts](main.ts) | window creation, IPC handlers, Flask spawn, port assignment, shutdown. Compiles to `dist-electron/main.js` via `tsconfig.electron.json`. |
| Preload | [preload.ts](preload.ts) | exposes a typed `window.electronAPI` to renderer via `contextBridge` (renderer has `nodeIntegration: false`, `contextIsolation: true`). Imports the contract from `src/types/electron-api.ts`. |
| ElectronAPI contract | [src/types/electron-api.ts](src/types/electron-api.ts) | single source of truth for the bridge surface. Augments `Window` via `declare global`. |
| Renderer entry | [src/index.tsx](src/index.tsx) | mounts React 18 under Redux `<Provider>` via `createRoot` |
| React app | [src/components/App.tsx](src/components/App.tsx) | example Flask `get<string>('example')` call on mount, ref-guarded for StrictMode |
| Typed Redux hooks | [src/state/hooks.ts](src/state/hooks.ts) | `useAppSelector` + `useAppDispatch` — components use these instead of plain `react-redux` hooks |
| Flask | [app.py](app.py) | single file, port from `sys.argv[1]`, binds 127.0.0.1, dev-only CORS for both `localhost:3000` and `127.0.0.1:3000` |
| Build/start scripts | [scripts/dispatch.ts](scripts/dispatch.ts) | command router for build/clean/package/start. Run via `tsx`. |

## How the three processes connect

1. `yarn start` → `tsx ./scripts/dispatch.ts start` → `Starter.developerMode()` ([scripts/start.ts](scripts/start.ts)) — kills port 3000, boots `vite` (host/port/strictPort/open configured in [vite.config.ts](vite.config.ts)), runs `tsc -p tsconfig.electron.json` to (re)compile `main.ts` + `preload.ts` to `dist-electron/`, boots `electron .`. Filters known-noisy stderr from both children.
2. Electron `dist-electron/main.js` (compiled from [main.ts](main.ts)) picks a free port in **3001–3999** via `get-port`, opens a frameless main window (loads `http://127.0.0.1:3000`) + a redux-themed loading window, and spawns `python app.py <port>`.
3. The renderer reads the port via `window.electronAPI.getPort()` (preload bridges to the sync IPC channel `get-port-number`) and calls Flask at `http://127.0.0.1:<port>`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iPzard/electron-react-python-template](https://github.com/iPzard/electron-react-python-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
