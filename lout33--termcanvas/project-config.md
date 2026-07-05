---
trigger: always_on
description: This file explains the local architecture of the `TermCanvas` app so future agents can extend it safely.
---

# AGENTS.md

This file explains the local architecture of the `TermCanvas` app so future agents can extend it safely.

## Product intent

This is a minimal infinite-canvas Electron app for spatial terminal workflows.

Each node is a real interactive terminal, not a text note. The product should stay simple and terminal-first.

## Architecture overview

### Main process — `main.js`

Responsibilities:

- create the Electron window
- own the terminal session registry
- spawn PTY clients through `node-pty`
- attach PTY clients to long-lived `tmux` sessions when `tmux` is available
- fall back to plain shell PTYs when `tmux` is unavailable
- validate that a renderer only talks to its own terminal sessions
- detach live tmux-backed sessions when a window closes
- permanently destroy a terminal session when the user closes a node
- own workspace folder import, refresh, watch, and preview access
- own app-session file persistence in Electron `userData`
- own file dialog and local file read/write for canvas JSON and app-data JSON import/export

Important rule:

**Real shell processes belong in main, never in the renderer.**

### Terminal identity model

The current session store is:

- `Map<terminalId, session>`

Important distinction:

- `terminalId` is the current renderer-to-main attachment id
- `sessionKey` is the stable terminal identity saved in app-session snapshots
- tmux session names are derived from `sessionKey`

Each session currently tracks:

- `ownerWebContentsId`
- `pty`
- `shellName`
- `cwd`
- `backend` (`"tmux"` or `"pty"`)
- `sessionKey`
- `tmuxSessionName`
- `isDisposing`

Detach semantics matter:

- closing the app/window should detach from tmux-backed sessions in normal runs
- clicking a node close button should kill the underlying session permanently
- smoke-test mode disables persistent relaunch behavior so tests do not leak sessions

### App-session persistence

The app now persists a normalized JSON snapshot under Electron `userData`.

That snapshot includes:

- canvases and active canvas
- viewport position and zoom
- terminal node layout, titles, cwd, maximize state, and stable `sessionKey`
- workspace folders, active folder, expanded directories, and file preview state
- sidebar and onboarding UI state

Normalization lives in `session_snapshot.js`.

### Preload bridge — `preload.js`

Responsibilities:

- expose the smallest safe API to the renderer
- translate renderer calls into IPC invocations/events
- expose narrow app-session, workspace, terminal, and import/export methods

Important rule:

**Do not expose raw `ipcRenderer`, Node APIs, or arbitrary shell execution hooks.**

Current exposed API includes:

- `loadAppSession()`
- `saveAppSession(payload)`
- `saveAppSessionFile(payload)`
- `openAppSessionFile()`
- `restoreWorkspaceSession(payload)`
- `getWorkspaceDirectoryState()`
- `openWorkspaceDirectory()`
- `refreshWorkspaceDirectory()`
- `activateWorkspaceFolder(folderId)`
- `reorderWorkspaceFolder(folderId, targetIndex)`
- `removeWorkspaceFolder(folderId)`
- `readWorkspaceFile(folderId, relativePath)`
- `createTerminal(payload)`
- `resolveTrackedTerminalCwds(terminalIds)`
- `writeTerminal(terminalId, data)`
- `resizeTerminal(terminalId, cols, rows)`
- `destroyTerminal(terminalId, options)`
- `saveCanvasFile(payload)`
- `openCanvasFile()`
- `onTerminalData(callback)`
- `onTerminalExit(callback)`
- `onTerminalCwdChange(callback)`
- `onWorkspaceDirectoryData(callback)`

### Renderer — `renderer.js`

Responsibilities:

- maintain viewport offset and zoom for the infinite canvas
- create terminal nodes in world coordinates
- host `xterm.js` instances inside node containers
- route keyboard input to the correct PTY client
- react to terminal output, exit, and cwd events from preload
- manage canvases, workspace drawer state, and file inspector state
- serialize and hydrate app-session state
- serialize canvas exports and restore imported canvases as new terminal nodes

Important rule:

**Renderer nodes are views plus layout state. They are not the owners of shell processes.**

Renderer node records now carry both:

- `terminalId`: current live attachment, nullable while detached or exited
- `sessionKey`: stable persisted identity for reconnecting to the same tmux-backed shell

## Current interaction model

- drag empty paper to pan the canvas
- modifier-wheel on empty paper zooms the canvas
- double-click empty paper to create a terminal node
- drag a terminal node by its header to move it
- resize nodes from edge and corner handles
- maximize a node into the board and restore it in place
- `Cmd+B` toggles the left drawer
- `Cmd+L` closes the file preview
- closing the app should preserve live tmux-backed terminals for reattach on relaunch
- clicking a terminal node close button should remove the node and kill that terminal session

## IPC contract

### Request/response

- `app-session:load`
- `app-session:save`
- `app-session:save-file`
- `app-session:open-file`
- `workspace-session:restore`
- `workspace-directory:state`
- `workspace-directory:open`
- `workspace-directory:refresh`
- `workspace-directory:debug-open`
- `workspace-folder:activate`
- `workspace-folder:reorder`
- `workspace-folder:remove`
- `workspace-file:read`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lout33/termcanvas](https://github.com/lout33/termcanvas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-05 -->
