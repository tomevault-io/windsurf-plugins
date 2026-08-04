---
trigger: always_on
description: Quartermaster is a **modular gaming toolkit hub** -- a desktop app that serves as a
---

# Quartermaster - Project Instructions

## Project Overview

Quartermaster is a **modular gaming toolkit hub** -- a desktop app that serves as a
central dashboard for game-specific tools, databases, and utilities. Think of it like
a workbench where each drawer is dedicated to a different game.

**Core ideas:**
- **Offline-first**: The app saves a local snapshot of all data. If the internet drops,
  everything keeps working. When connectivity returns, it syncs only what changed.
- **Modular**: Each game is a self-contained plugin. You can enable or disable them
  independently. Adding a new game never breaks existing ones.
- **Desktop-first**: Ships as a native desktop app via Tauri. Mobile support comes later
  using the same Tauri v2 codebase.
- **Dark theme by default**: Because we are not animals.
- **Personal use first**: Build for yourself, polish for public release later.

**Target games (in priority order):**
1. Escape from Tarkov -- first module, serves as the testbed for the plugin system
2. Elite Dangerous
3. Star Citizen

---

## Tech Stack

| Layer | Technology | Why |
|-------|-----------|-----|
| Desktop shell | **Tauri v2** | Lightweight native wrapper, Rust backend, future mobile support |
| Frontend | **React 19 + TypeScript** | Component-based UI, strong typing catches bugs early |
| Styling | **Tailwind CSS v4** | Utility-first, easy dark theme, fast iteration |
| State management | **Zustand** | Simple, minimal boilerplate, works great with React |
| Local DB (browser) | **Dexie.js (IndexedDB)** | Offline snapshot storage, fast reads from the browser side |
| Local DB (native) | **SQLite via Tauri plugin** | Persistent storage on disk, survives app reinstalls |
| API client | **urql or graphql-request** | Lightweight GraphQL client for Tarkov.dev API |
| Build tool | **Vite** | Fast dev server, comes with Tauri scaffolding |

### Why two databases?

This is a common pattern in offline-first apps. Think of it this way:

- **Dexie/IndexedDB** lives inside the browser/webview. It is fast for the UI to read
  from and is where the "current working data" lives. The React app talks directly to it.
- **SQLite** lives on the native filesystem via Tauri's Rust backend. It is the durable
  "source of truth" that persists even if the webview cache gets cleared. It also handles
  things the browser cannot do, like file exports or background sync.

Data flows: `API --> SQLite (persist) --> Dexie (UI cache) --> React components`

---

## Architecture Overview

### The Big Picture

```
+------------------------------------------------------------------+
|                        QUARTERMASTER APP                          |
|                                                                   |
|  +------------------+   +-------------------------------------+  |
|  |                  |   |           GAME MODULES              |  |
|  |    HUB CORE      |   |                                     |  |
|  |                  |   |  +----------+  +---------+  +-----+ |  |
|  |  - Shell / Nav   |   |  | Tarkov   |  | Elite   |  | SC  | |  |
|  |  - Theme engine  |   |  | Module   |  | Module  |  | Mod | |  |
|  |  - Plugin loader |   |  |          |  |         |  |     | |  |
|  |  - Sync engine   |   |  +----------+  +---------+  +-----+ |  |
|  |  - Settings      |   |                                     |  |
|  |  - Offline mgr   |   +-------------------------------------+  |
|  |                  |                                             |
|  +------------------+                                             |
|                                                                   |
|  +------------------------------------------------------------+  |
|  |                    SHARED SERVICES                          |  |
|  |  Data layer (Dexie + SQLite) | API client | Network monitor |  |
|  +------------------------------------------------------------+  |
|                                                                   |
|  +------------------------------------------------------------+  |
|  |                    TAURI BACKEND (Rust)                     |  |
|  |  SQLite plugin | File system | System tray | Auto-updater  |  |
|  +------------------------------------------------------------+  |
+------------------------------------------------------------------+
```

**How to read this:**

1. **Hub Core** is the app shell -- navigation, settings, theme, the plugin loader that
   mounts/unmounts game modules. It exists regardless of which games are enabled.
2. **Game Modules** are independent plugins. Each one registers itself with the hub and
   gets a navigation entry. Disabling a module removes it from the UI entirely.
3. **Shared Services** are utilities that any module can use: the database layer, the
   network-aware API client, the offline/online detection.
4. **Tauri Backend** is the Rust layer that provides native OS capabilities.

### What is a "module"?

A module is a folder that exports a standard shape:

```typescript
// Every game module exports this contract
export interface GameModule {
  id: string;              // "tarkov", "elite-dangerous", "star-citizen"
  name: string;            // Display name
  version: string;         // Semver

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DrBaron420/Quartermaster](https://github.com/DrBaron420/Quartermaster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
