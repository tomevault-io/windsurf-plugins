---
trigger: always_on
description: > This document is written for AI coding agents who have no prior knowledge of the project. It describes the actual structure, technology stack, runtime behavior, and development conventions observed in the source tree.
---

# AGENTS.md — DevilConnection Browser Port

> This document is written for AI coding agents who have no prior knowledge of the project. It describes the actual structure, technology stack, runtime behavior, and development conventions observed in the source tree.

## Project overview

This repository is a **browser port of the visual novel *DevilConnection* (恶魔连结 / デビルコネクション)**. It is built on top of the **TyranoScript** visual novel engine (KAG format) and runs as a static HTML5 application in a web browser.

The repository name in `package.json` is `devil-connection-browser`. The original project used Electron + Steamworks; the current browser version replaces the native preload APIs with a browser shim so the same TyranoScript assets can run without Electron or Node.js.

Key facts:

- **No build step is required** to run the game in a browser.
- **No server-side component** exists.
- All game logic is written in TyranoScript scenario files (`.ks`) and vanilla JavaScript.
- The game is intended to be served as static files (`index.html`, `data/`, `tyrano/`).

## Technology stack

- **Frontend runtime**: HTML5, CSS, vanilla JavaScript (ES5-style IIFE modules), jQuery 3.6.0, jQuery UI, jQuery Migrate.
- **Visual novel engine**: TyranoScript (KAG script format, `*.ks`).
- **Audio**: Howler.js + HTML5 `<audio>` for autoplay policy handling.
- **Dialogs / overlays**: SweetAlert2, Remodal, Alertify.js (legacy).
- **Utilities**: JSZip, html2canvas, jsrender, LZString, jsQR, APNG support (`apng.js`, `blob.js`).
- **3D support (mostly unused)**: Three.js and its loaders (the config has `use3D=false`).
- **Text effects**: textillate, animate.css, lettering.js, touchSwipe.
- **Node usage**: only for a small `serve` script in `package.json`; no bundler, transpiler, or test framework is present.
- **Legacy native shell**: `_electron_legacy/` contains the original Electron main/preload/Steam integration, but the current `index.html` does not load it.

## Key configuration files

| File | Purpose |
|------|---------|
| `package.json` | Project metadata. Only defines `scripts.serve` which runs `npx -y serve .`. |
| `index.html` | Entry point. Loads all libraries, the browser API shim, and the Tyrano engine, then waits for the user click on `#tyrano_click_to_start` before initializing the game. |
| `data/system/Config.tjs` | TyranoScript game configuration (screen size, text speed, default volumes, save settings, etc.). Generated/managed by TyranoBuilder. |
| `data/system/KeyConfig.js` | Keyboard/mouse/gesture bindings for the game. |
| `tyrano/lang.js` | In-game text strings. The current file contains Simplified Chinese UI text (with some Japanese names) for this port. |
| `data/scenario/first.ks` | First scenario loaded by TyranoScript after initialization. It loads system macros and jumps to `title_screen.ks`. |
| `browser_api.js` | Browser-only `window.api` shim (file system, storage, dialogs, fullscreen, etc.). |
| `electron_latest.js` | Browser adaptation overrides for TyranoScript core behavior (save integration, patch disabling, `web`/`close` tags). |

## Directory layout

```
.
├── index.html              # Application entry point
├── package.json            # Minimal Node metadata / serve script
├── browser_api.js          # Browser shim for Electron preload APIs
├── electron_latest.js      # Tyrano runtime browser patches
├── favicon.ico
├── data/                   # Game assets and scripts
│   ├── bgimage/            # Background images
│   ├── fgimage/            # Foreground / character images
│   ├── image/              # UI / misc images
│   ├── sound/              # Sound effects
│   ├── bgm/                # Background music
│   ├── video/              # Movies
│   ├── others/             # Custom JavaScript, plugins, and game data
│   │   ├── plugin/         # TyranoBuilder-style plugins (init.ks + JS)
│   │   └── *.js            # Game-specific helpers (master_data, loading overlay, etc.)
│   ├── scenario/           # Main KAG scenario files
│   │   ├── system/         # System macros and auto-generated backups (`_*.ks`)
│   │   └── *.ks            # Story scripts
│   └── system/             # Tyrano system files (Config.tjs, KeyConfig.js)
├── tyrano/                 # TyranoScript engine
│   ├── tyrano.js           # Core bootstrap
│   ├── tyrano.base.js      # Base layer / screen scaling
│   ├── libs.js             # jQuery / utility extensions
│   ├── lang.js             # In-game strings
│   ├── libs/               # Third-party libraries
│   ├── plugins/kag/        # KAG engine plugin (kag.js, tags, parser, menu, etc.)
│   ├── css/                # Engine styles
│   └── images/             # Engine default images
└── _electron_legacy/       # Original Electron + Steamworks files (not used by browser build)
```

## Runtime architecture

1. The browser loads `index.html`.
2. `browser_api.js` runs first and creates `window.api`, a browser-compatible replacement for the Electron preload API (storage, file dialogs, fullscreen, etc.).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lllhhh2282/devilconnection-web](https://github.com/lllhhh2282/devilconnection-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
