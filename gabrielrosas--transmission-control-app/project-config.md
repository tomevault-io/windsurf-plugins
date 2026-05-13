---
trigger: always_on
description: Context and conventions for AI agents working on this repository.
---

# AGENTS.md

Context and conventions for AI agents working on this repository.

## What this app is

Electron desktop app that acts as a side-docked control panel for live broadcasts. It controls **PTZ cameras over ONVIF** and the **OBS Studio scene switcher over OBS WebSocket**, and ties them together: clicking a preset moves the camera *and* swaps the OBS scene.

Target user: streamers / AV operators (church, studio, event production). UI text is in **Portuguese (pt-BR)**.

## Stack

- **Electron 38** (main + preload + renderer) scaffolded with `electron-vite`
- **React 19 + TypeScript** in the renderer
- **Tailwind v4** (`@tailwindcss/vite`) + Radix primitives + lucide-react + react-hot-toast
- **Zustand** for client state, **TanStack Query** for async/server state
- **react-hook-form + Zod** for forms and validation
- **Firebase** (Auth + Firestore) for per-user config sync
- **`onvif`** for PTZ control, **`obs-websocket-js`** for OBS

## Layout

```
src/
  main/           # Electron main process
    index.ts      # window creation, IPC wiring, menu
    Cam.ts        # ONVIF PTZ controller + CamMock for dev
    ImageCache.ts # IPC for caching base64 images under userData/images/
  preload/
    index.ts      # contextBridge: window.ptz, window.clipboard, window.imageCache
    index.d.ts    # global Window typings
  renderer/src/
    main.tsx              # router root (HashRouter)
    providers/            # AuthProvider, OBSProvider, QueryClientProvider
    hooks/
      firebase.ts         # auth + firestore init, useAuth zustand store
      config.ts           # firestore-synced user config (zustand)
      obs.ts              # OBS WebSocket singleton + zustand store
      ptz/                # PTZ contexts and hooks
      utils/confirm.tsx   # confirm dialog hook
    routes/
      home/               # main panel: PTZ cards + OBS card
      settings/           # /settings/{obs,ptz,history}
    components/           # Box, Button, Dialog, ContextMenu, form/, containers/, ...
    schemas/              # Zod schemas (CameraPTZ, OBSConfig, Overlayer)
    libs/cn.ts            # tailwind-merge helper
```

Path alias: `@renderer` → `src/renderer/src` (configured in [electron.vite.config.ts](electron.vite.config.ts) and `tsconfig.web.json`).

## Architecture notes

### Window
[src/main/index.ts](src/main/index.ts:18) creates a fixed-width 400px window pinned to the right edge of the primary display, at full work-area height. It is meant to sit alongside OBS, not on top of it. Hardware acceleration is disabled (`app.disableHardwareAcceleration()`).

### IPC surface
Four preload bridges, all in [src/preload/index.ts](src/preload/index.ts):
- `window.ptz`: `init`, `getPresets`, `goto`, `getPosition`, `onConnected`, `onLogs`
- `window.clipboard`: `writeText`
- `window.imageCache`: `save`, `get`, `clear`, `clearFolder` — files written to `app.getPath('userData')/images/<folder>/<filename>.cache`
- `window.overlays`: `put({ apiUrl, payload })` — proxies an HTTP PUT to the overlays.uno (Singular) API from the main process; avoids CORS issues that block direct fetch from the renderer

When you change the IPC surface, update **all three**: the handler in `main/`, the preload definition, and the typings in [src/preload/index.d.ts](src/preload/index.d.ts).

### PTZ
[src/main/Cam.ts](src/main/Cam.ts) wraps `onvif/promises` with a `Cam` class and a `CamStore` registry keyed by config id. There is also a `CamMock` enabled when `PTZ_CAM_DEV=true` (set in the `dev` npm script via `cross-env`). The mock fakes 30 presets and a moving position. Use it for renderer work without real cameras.

### OBS
[src/renderer/src/hooks/obs.ts](src/renderer/src/hooks/obs.ts) holds a module-level `OBSConnectionStore` singleton and a `useOBS` zustand store. It listens to OBS events (`CurrentProgramSceneChanged`, `CurrentPreviewSceneChanged`, `SceneListChanged`) and reflects them into the store. There is a `delay(2000)` after `ConnectionOpened` before fetching scenes — leave it alone unless you confirm OBS is ready earlier.

### PTZ ↔ OBS coupling
The interesting logic lives in [`useInitPTZPreset`](src/renderer/src/hooks/ptz/hooks.ts) (`gotoPresetBase`):
1. If the camera is currently on Program and the user picks a different preset, switch Program to `axSceneId` (auxiliary scene) **first** so viewers don't see the camera moving.
2. Move the camera (`window.ptz.goto`).
3. Wait `transitionTime` ms, then read the new position and (optionally) push the camera's scene back to Program.

`axSceneId` exists specifically to mask camera transitions. Don't refactor it away.

### Config persistence
Config is synced per `auth.uid` via Firestore `onSnapshot` ([src/renderer/src/hooks/config.ts](src/renderer/src/hooks/config.ts:50)). The `Config` shape is `{ obsConfig, cameraPTZConfig, presetsAlias, presetsHidden }`:
- `presetsAlias: Record<string, string>` — flat map keyed by `${cameraId}-${presetId}` → display name (renames in the PTZ panel).
- `presetsHidden: Record<string, string[]>` — per-camera array of preset IDs hidden from the panel.

Both go through `setConfig` like any other config field, so each rename/hide creates a new history version (intentional — the user accepted the trade-off).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gabrielrosas/transmission-control-app](https://github.com/gabrielrosas/transmission-control-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
