---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Jitsi Meet Electron is a desktop application for Jitsi Meet built with Electron. It provides features like E2E encryption, screen sharing, always-on-top window, auto-updates, and deeplink support for conferencing.

## Development Commands

### Setup
- **Install dependencies**: `npm install`
  - Requires Node.js 24+ (see `.nvmrc`)
  - Linux requires: `libx11-dev zlib1g-dev libpng-dev libxtst-dev`
  - Windows requires: `windows-build-tools` (global install)

### Development
- **Start in development mode**: `npm start`
  - Runs esbuild for the main, preload, and renderer bundles, then starts concurrent watch + electron
  - Opens WebRTC internals window automatically in dev mode
- **Start with DevTools open**: `SHOW_DEV_TOOLS=true npm start` or `npm start -- --show-dev-tools`
- **Watch renderer changes**: `npm run watch` (runs automatically with `npm start`)

### Building
- **Lint code**: `npm run lint` (ESLint over `.js`, `.ts`, `.tsx`)
- **Fix lint issues**: `npm run lint-fix`
- **Type-check**: `npm run type-check` (`tsc --noEmit`; esbuild itself does not type-check)
- **Build for production**: `npm run build` (runs `type-check` then esbuild for the main, preload, and renderer bundles)
- **Create distribution**: `npm run dist` (runs build then electron-builder)
- **Clean build artifacts**: `npm run clean`

The codebase is written in **TypeScript** (`.ts` / `.tsx`). esbuild strips types during bundling but does not check them, so type errors are caught by `npm run type-check` (and the type-aware ESLint pass), which `npm run build` runs first. The only remaining `.js` files are build tooling (`esbuild.js`, `.eslintrc.js`) and the vendored, pre-bundled `app/features/conference/external_api.js` (typed by an adjacent `external_api.d.ts`). Ambient type declarations live in `types/` (`global.d.ts` for the `window.jitsiNodeAPI` global and `process.mas`; `modules.d.ts` for untyped deps and `*.svg`/`*.png`/`*.css` imports). Shared redux/state interfaces live in `app/types.ts` (`IState`, `IConference`, etc.).

### CI Workflow
The CI runs on push/PR to master:
1. `npm ci` - Clean install
2. `npm run lint` - Linting (Linux only in CI)
3. `npm run dist` - Build distributables for all platforms

## Architecture

### Dual-Process Architecture
The application follows Electron's main/renderer process model:

**Main Process** (`main.ts`):
- Entry point for Electron
- Manages BrowserWindow lifecycle
- Handles protocol calls (`jitsi-meet://` deeplinks)
- Configures security policies (CSP, file:// blocking, redirect filtering)
- Integrates `@jitsi/electron-sdk` features:
  - Remote control (controlled by `ENABLE_REMOTE_CONTROL` flag)
  - Always-on-top window
  - Screen sharing
  - Power monitor
  - Popup configuration
- Auto-update handling via `electron-updater`
- IPC communication with renderer

**Renderer Process** (`app/` directory):
- React application bundled via esbuild
- Contains all UI components and business logic
- Uses `@jitsi/electron-sdk` via preload script (`app/preload/preload.ts`)
- Communicates with main process via IPC

### Build System
A single esbuild script (`esbuild.js`) builds all three bundles into `build/`.
Run it directly with target/flag arguments:
- `node ./esbuild.js` — production build of everything
- `node ./esbuild.js --dev` — development build (unminified, inline sourcemaps)
- `node ./esbuild.js renderer --dev --watch` — watch/rebuild the renderer (this is `npm run watch`)
- `node ./esbuild.js main` — build only main + preload

**Main config** (`main` target):
- `platform: 'node'`, `format: 'cjs'`, `target: 'node22'`
- Bundles `main.ts` and `app/preload/preload.ts`
- Externalizes `electron` plus the runtime dependencies that ship in
  `node_modules` (`@jitsi/electron-sdk`, `electron-debug`, `electron-reload`);
  everything else (devDependencies used by the main process) is bundled in

**Renderer config** (`renderer` target):
- `platform: 'browser'`, `format: 'iife'`, `target: 'chrome120'`
- Entry: `app/index.tsx` → `build/app.js` (+ `build/app.css` from imported CSS)
- JSX lives in `.tsx` files; esbuild compiles TS/TSX natively (classic runtime, React in scope)
- `process.env.NODE_ENV` is injected via `define`; `global` is polyfilled via banner
- Custom esbuild plugins replace the old webpack loaders/plugins:
  - `svgr` — turns imported `.svg` files into React components (uses `@svgr/core`,
    replacing `@svgr/webpack`; options `dimensions: false`, `expandProps: 'start'`)
  - `external-api` — keeps the vendored, pre-bundled `external_api.js` usable
    (strips its dangling sourceMappingURL; stands in for webpack's `noParse`)
  - `html` — renders `index.html`/`meeting.html` from the `app/` templates and
    injects the `<link>`/`<script>` tags (replaces `HtmlWebpackPlugin`)

### Feature-Based Architecture
Code is organized by feature domain under `app/features/`:

```
app/features/
├── app/          - Root App component and routing
├── conference/   - Conference iframe and JitsiMeetExternalAPI integration
├── config/       - Application configuration constants

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jitsi/jitsi-meet-electron](https://github.com/jitsi/jitsi-meet-electron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
