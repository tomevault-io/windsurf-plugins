---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What it is

Drawverlay is an Electron desktop app that renders an always-on-top overlay for drawing on screen during presentations and recordings. Target platforms: macOS (primary), Windows (future).

## Commands

```bash
npm install        # install dependencies (electron + electron-builder)
npm start          # launch the app
npm run kill       # force-kill stuck electron process
npm run dist:mac   # build macOS distributable (DMG + ZIP, arm64 + x64)
npm run dist       # build for current platform
```

There is no test suite yet.

## Architecture

Root files + `components/` folder for all window assets:

| File | Role |
|------|------|
| `main.js` | Electron main process — windows, global shortcuts, IPC handlers, settings persistence |
| `preload.js` | Context bridge — exposes `window.drawverlay` API to renderer via `contextBridge` |
| `components/renderer.html` | Overlay shell — transparent canvas + bg div + cursor disc |
| `components/renderer.css` | Overlay styles |
| `components/renderer.js` | All drawing logic — canvas 2D, brush/eraser/rect/line/grid, easing loop, mouse events |
| `components/menu.html` | Tray panel — four tabs (Tools / Projects / Settings / Shortcuts) |
| `components/menu.css` | Menu styles |
| `components/menu.js` | Menu logic — color pickers, palette, settings controls |
| `components/tooltip.html` | Tooltip overlay window |
| `components/tooltip.css` | Tooltip styles |
| `components/tooltip.js` | Tooltip logic |
| `web/` | Astro marketing website — `npm run dev` / `npm run build` inside this folder |
| `web/src/` | Astro source — components, layouts, pages |
| `web/src/components/DrawingBackground.astro` | Embeds full drawing app (canvas + menu) in the hero background |
| `web/public/components` | Symlink → `../../components` — serves component JS/CSS at `/components/` |
| `web/public/bridge.js` | Web bridge — implements `window.drawverlay` without Electron (CustomEvents + localStorage) |

## Key implementation details

**Overlay window** (`main.js`)
- `transparent: true`, `frame: false`, `alwaysOnTop: true`, `hasShadow: false`, `focusable: false`
- `setAlwaysOnTop(true, 'screen-saver')` + `setVisibleOnAllWorkspaces` so it sits above full-screen apps and the macOS menu bar
- Sized to `workArea` of the target display (not `bounds`) — avoids bottom-overflow on macOS
- `app.dock.hide()` on macOS — utility overlay, not a regular app
- `focusable: false` means the overlay never steals focus, but it also means **keyboard events never fire in `renderer.js`** — handle keys via global shortcuts in `main.js` or by reading `e.shiftKey` from mouse events

**Menu window** (`main.js`)
- Separate `BrowserWindow` (`menu.html`), shown/hidden on tray click, right-click on canvas, or Escape key
- Draggable via manual IPC drag (`menu:drag-start` / `menu:drag`) — `-webkit-app-region: drag` was removed because it prevents CSS cursor changes
- Height is resized dynamically via `menu:resize` IPC whenever tab content changes
- `showMenu(pos?)` clamps position using `screen.getDisplayNearestPoint(pos).bounds` — always use this helper, never `setPosition` directly, to stay within the correct display

**Passthrough mode** (`main.js` ↔ `renderer.js`)
- `win.setIgnoreMouseEvents(!isVisible || isPassthrough, { forward: true })` — combined condition; overlay is always non-interactive when hidden OR in passthrough. `forward: true` keeps mousemove flowing for cursor tracking. Never call `win.show/hide()` — use `win.setOpacity(isVisible ? 1 : 0)` + the combined `setIgnoreMouseEvents` call instead (both `setVisible` and `setPassthrough` must apply this same combined condition)

**Global shortcuts**
- Shortcuts are **user-configurable** — stored in `settings.json` under `shortcuts`; merged with `DEFAULT_SHORTCUTS` on load via `{ ...DEFAULT_SHORTCUTS, ...data.shortcuts }`
- Default bindings: `Cmd+Shift+X` (visible), `Cmd+Shift+C` (passthrough), `Cmd+Shift+B` (background), `Cmd+Shift+Backspace` (eraseAll), `Cmd+Z` / `Cmd+Shift+Z` (undo/redo), `B/E/R/L/G` (tools), `S` (sizing), `Escape` (menu)
- `globalShortcutHandlers` map holds handlers for the four non-single-key global shortcuts (`toggleVisible`, `togglePassthrough`, `toggleBackground`, `eraseAll`); registered via `registerGlobalShortcut(action)`
- `registeredGlobalAccelerators` object tracks currently registered accelerator per action — used by `applyShortcutChange` to unregister the old one before registering the new
- `registeredUndoAccelerator` / `registeredRedoAccelerator` track undo/redo separately; re-registered by `updateUndoShortcuts()` which is called from both `setVisible` and `setPassthrough`
- `applyShortcutChange(action, newAccelerator)` — live-swaps one shortcut; routes to `registerGlobalShortcut`, `updateUndoShortcuts`, or `forceUpdateSingleKeyShortcuts` based on action category; saves settings; broadcasts `shortcuts-changed` to menu
- **Single-key shortcuts** (tool keys, S, Escape) tracked in `registeredSingleKeyAccelerators` array; `forceUpdateSingleKeyShortcuts()` tears down and rebuilds them (needed after a shortcut change because `singleKeyShortcutsActive` state must be reset)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brunosimon/drawverlay](https://github.com/brunosimon/drawverlay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
