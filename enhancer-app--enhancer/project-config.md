---
trigger: always_on
description: Enhancer is a Manifest V3 browser extension that adds features to Twitch and Kick streaming platforms. It's built with TypeScript, Preact, styled-components, and uses Bun as the package manager with Vite as the build tool.
---

# AGENTS.md — Enhancer Browser Extension

## Project Overview

Enhancer is a Manifest V3 browser extension that adds features to Twitch and Kick streaming platforms. It's built with TypeScript, Preact, styled-components, and uses Bun as the package manager with Vite as the build tool.

## Build & Run

| Command | Description |
|---------|-------------|
| `bun run dev` | Concurrent typecheck:watch + vite build --watch + dev server (port 3360) |
| `bun run build` | Production: biome lint + typecheck + vite build (minified) |
| `bun run typecheck` | `tsc --noEmit` |
| `bun run typecheck:watch` | `tsc --noEmit --watch` |

Always run `bun run typecheck` and `npx @biomejs/biome check src/` after making changes.

## Directory Structure

```
src/
  index.ts                    # Main-world entry point (injected into page via <script> tag)
  inject.ts                   # Content script that creates the <script> tag
  platforms/
    twitch/                   # All Twitch-specific code
      twitch.platform.ts      # Platform orchestration, module registration
      twitch.module.ts        # TwitchModule base class (adds twitchUtils, twitchApi)
      twitch.constants.ts     # Default settings, asset paths
      modules/                # Twitch feature modules (~23 modules)
    kick/                     # All Kick-specific code (mirrors twitch/ structure)
  shared/
    apis/                     # EnhancerApi — external backend API client
    components/               # Reusable Preact UI components
    event/                    # EventEmitterFactory (wraps nanoevents)
    logger/                   # Colored console Logger
    module/                   # Module base class + applier pattern
      applier/                # SelectorModuleApplier, EventModuleApplier
      helpers/                # SettingsHelper (shared settings UI logic)
    platform/                 # Platform base class
    settings/                 # SettingsCache — local cache with broadcast sync
    storage/                  # StorageRepository (localStorage wrapper)
    utils/                    # CommonUtils, ReactUtils, UtilsRepository
    worker/                   # Worker architecture (bridge, background, databases, handlers)
      database/               # Abstract Database base class (IndexedDB)
      settings/               # SettingsDatabase + UpdateSettingsHandler
      watchtime/              # WatchtimeDatabase + WatchtimeAccumulator + handlers
  types/                      # All type definitions (never co-located with implementation)
    platforms/
      common.events.ts        # Shared event signatures (extension:start, settings-refresh, etc.)
      twitch/                 # TwitchEvents, TwitchSettings, TwitchStorage, TwitchApiTypes
      kick/                   # KickEvents, KickSettings, KickStorage, KickApiTypes
    shared/
      module/                 # ModuleConfig, ModuleApplierConfig (discriminated union)
      worker/                 # WorkerApiActions, WorkerBroadcast, payload/response types
      components/             # SettingCategory, SettingDefinition, SettingsProps
```

## Architecture: Three-Layer Communication

The extension spans three JavaScript contexts that share the DOM but have separate JS environments:

```
MAIN WORLD (index.ts / Platform)
    ↕ CustomEvents on <enhancer-bridge> DOM element
CONTENT SCRIPT (worker.bridge.ts)
    ↕ chrome.runtime.sendMessage / chrome.runtime.onMessage
BACKGROUND SCRIPT (worker.background.ts)
    ↕ IndexedDB (SettingsDatabase, WatchtimeDatabase)
```

### How it works

1. **Main world** (`index.ts`): Creates `WorkerService` which creates a `<enhancer-bridge>` DOM element. Sends messages via `enhancer-message` CustomEvents, receives responses via `enhancer-response` CustomEvents.

2. **Content script** (`worker.bridge.ts`): Uses MutationObserver to find `<enhancer-bridge>`, forwards CustomEvents to/from `chrome.runtime.sendMessage`. Dispatches `enhancer-bridge-ready` when connected.

3. **Background script** (`worker.background.ts`): Routes messages to `MessageHandler` instances via `HandlerRegistry`. Manages IndexedDB databases and the `WatchtimeAccumulator`.

### Critical: Bridge Readiness

`WorkerService.start()` awaits a `enhancer-bridge-ready` CustomEvent from the bridge before resolving. This gates `settingsCache.initialize()` so the first message is never lost. Never remove this readiness check.

### Adding a new worker action

1. Define the action type and payload/response in `src/types/shared/worker/worker.types.ts` (`WorkerApiActions`)
2. Create a handler class extending `MessageHandler` in `src/shared/worker/`
3. Register it in `HandlerRegistry`
4. Call `workerService.send("actionName", payload)` from the main world

## Module System

Every feature is a module. Modules extend `Module<Events, Storage, Settings>` (or platform-specific `TwitchModule` / `KickModule`).

### Module lifecycle

1. **Construction** — receives dependencies via constructor (emitter, settings cache, utils, etc.)
2. `setup()` — creates scoped Logger (`module:{name}`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [enhancer-app/enhancer](https://github.com/enhancer-app/enhancer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
