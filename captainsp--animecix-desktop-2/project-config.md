---
trigger: always_on
description: Electron desktop application for AnimeciX — anime streaming, downloading, and offline playback.
---

# AnimeciX Desktop App

Electron desktop application for AnimeciX — anime streaming, downloading, and offline playback.

## Tech Stack

- **Runtime:** Electron 41 + Electron Forge 7 (Vite plugin)
- **Language:** TypeScript 5.9 (strict mode, `noImplicitAny: true`)
- **UI:** React 19 (player page + library page), Angular (website shell via animecix.tv)
- **Video:** Vidstack React + hls.js + JASSUB (ASS/SSA subtitle rendering)
- **Database:** better-sqlite3 (synchronous SQLite, WAL mode)
- **Build:** 5 Vite configs (main, preload, renderer, player, library)
- **Testing:** Vitest 4
- **Linting:** ESLint with TypeScript + import plugins

## Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│  Main Process (src/main.ts)                         │
│  ┌───────────┐ ┌──────────┐ ┌───────────────────┐  │
│  │ Storage   │ │ Download │ │ StreamCache       │  │
│  │ (SQLite)  │ │ Queue    │ │ (transparent +    │  │
│  │           │ │          │ │  explicit caching) │  │
│  └───────────┘ └──────────┘ └───────────────────┘  │
│  ┌───────────┐ ┌──────────┐ ┌───────────────────┐  │
│  │ Discord   │ │ Updater  │ │ Library Manager   │  │
│  │ RPC       │ │ Service  │ │ (BrowserView)     │  │
│  └───────────┘ └──────────┘ └───────────────────┘  │
│  ┌───────────┐ ┌──────────┐ ┌───────────────────┐  │
│  │ AdBlocker │ │ Header   │ │ TrayManager       │  │
│  │           │ │ Rewriter │ │                   │  │
│  └───────────┘ └──────────┘ └───────────────────┘  │
├─────────────────────────────────────────────────────┤
│  Preload (src/preload.ts)                           │
│  contextBridge → window.animecix + window.animecixAPI│
├─────────────────────────────────────────────────────┤
│  Renderer: animecix.tv (Angular website)            │
│  ┌─────────────────────────────────────────────┐    │
│  │  Player iframe (tau-player://)  — React     │    │
│  │  Library overlay (animecix-library://) — React│   │
│  └─────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────┘
```

### Custom Protocol Schemes

| Protocol | Purpose | Handler |
|---|---|---|
| `tau-player://` | Serves local video player assets (Vidstack + JASSUB) | `src/player/tau-protocol.ts` |
| `animecix-offline://` | Serves cached/downloaded video files | `src/offline/offline-protocol.ts` |
| `animecix-library://` | Serves library overlay React app | `src/library/library-protocol.ts` |
| `animecix://` | Deep link protocol for Google auth callback | `src/auth/deep-link.ts` |

### IPC Communication Pattern

```
animecix.tv (renderer)
    │
    ├─ window.animecix.*  ──────► ipcRenderer.invoke() ──► ipcMain.handle()
    │   (preload bridge)
    │
    ├─ postMessage ◄────────────► Player iframe (tau-player://)
    │   (player iframe CANNOT access window.animecix — only postMessage)
    │
    └─ window.animecixAPI.updater ──► ipcRenderer (updater channels)
```

**Critical rule:** The player iframe runs under `tau-player://` origin. It has NO access to
Electron's `ipcRenderer`. All player↔main communication goes through the website as a bridge:
`player iframe → postMessage → animecix.tv → IPC → main process`.

## Directory Structure

```
src/
├── main.ts              # Electron main process entry point
├── preload.ts           # contextBridge API (AnimecixAPI contract)
├── renderer.ts          # Renderer process entry
├── auth/                # Deep link protocol (Google login callback)
├── cache/               # StreamCache (transparent + explicit), HlsMuxer, CacheEvictor
├── download/            # Multi-threaded downloader, queue, tray, IPC handlers
├── integrations/        # Discord Rich Presence
├── library/             # Offline library manager (BrowserView overlay)
├── library-page/        # React app for offline library UI
├── network/             # Ad blocker, request interception, CDN header rewriting
├── offline/             # animecix-offline:// protocol handler
├── player/              # tau-player:// protocol handler
├── player-page/         # React app for video player (Vidstack + JASSUB)
├── storage/             # SQLite StorageService + schema
├── types/               # TypeScript type definitions
├── updater/             # Auto-update service + in-app banner
└── window/              # BrowserWindow creation, lifecycle, IPC
```

## Code Rules

### Language

- All code (variables, functions, classes, comments, commit messages) MUST be in **English**.
- Turkish is ONLY allowed in user-facing UI strings (button labels, notifications, messages).
- JSDoc comments and inline comments must be in English.

### File Organization

- Each domain has its own directory: `download/`, `cache/`, `storage/`, etc.
- IPC handlers go in `<domain>.ipc.ts` files (e.g., `download.ipc.ts`).
- Type definitions go in `<domain>.types.ts` or `src/types/`.
- Protocol handlers go in `<domain>-protocol.ts` files.
- React pages (player, library) are self-contained in `<name>-page/` directories.

### IPC Rules

1. **Never expose `ipcRenderer` directly.** All IPC must go through `preload.ts` → `contextBridge`.
2. IPC channel names follow `domain:action` pattern (e.g., `download:start`, `window:minimize`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CaptainSP/animecix-desktop-2](https://github.com/CaptainSP/animecix-desktop-2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
