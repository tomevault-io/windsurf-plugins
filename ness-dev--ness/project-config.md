---
trigger: always_on
description: This file is read automatically by Claude Code at the start of every session.
---

# Ness — repo overview for Claude

This file is read automatically by Claude Code at the start of every session.
It documents the project structure and the conventions used here.

## What this app is

Ness is a macOS Electron app that manages multiple Claude Code instances
across git worktrees. The user runs many parallel Claude sessions, and Ness
gives them a single window with a sidebar of worktrees, terminal tabs per
worktree (Claude + raw shells), changed-files panel, PR status, and hotkey
navigation.

## Stack

- **Electron** main process + **React 19 / TypeScript** renderer
- **electron-vite** for the dev/build pipeline
- **xterm.js** + **node-pty** for terminals
- **Tailwind CSS v4** (CSS-imported, no PostCSS plugin) for styling
- **lucide-react** v1.x for icons (note: brand icons like `Github` are NOT exported in this version — use `GitPullRequest` etc.)
- **electron-builder** for packaging, signed with the user's personal Developer ID, notarized
- **electron-updater** for OTA updates from GitHub releases
- **`@anthropic-ai/claude-code`** is bundled as a dep (pinned native binary) and used by Chat tabs (internally `json-mode`) only. Terminal tabs (internally xterm-hosted) continue to spawn the user's PATH `claude` so power users on bleeding-edge / beta builds keep that experience. Both share `~/.claude/` for auth + MCP config.

## Architecture (read this before touching state)

This app went through a large refactor where **the main process owns all
shared state**, the renderer is a thin view layer, and a single transport
(currently Electron IPC, future: WebSocket) carries both state events
and side-effect signals. If you find yourself adding a `useState` to
hold a value that any other client of this workspace would also want,
you're doing it wrong — that value belongs in a slice.

### The store + slice pattern

State is partitioned into **slices** under `src/shared/state/`. Each
slice has:
- A `State` interface (the data shape)
- An `Event` discriminated union (the mutations)
- A `reducer(state, event) → state` pure function
- An `initial<Slice>` constant
- A test file with one test per event variant

Current slices: `settings`, `prs`, `onboarding`, `hooks`, `worktrees`,
`terminals` (which also owns `panes` and `lastActive`), `updater`,
`repoConfigs`. Adding a new piece of shared state means picking the
right slice (or making a new one) and editing the reducer + event union.

### How a state mutation flows end-to-end

1. **Renderer**: user clicks something. The handler calls a thin IPC
   method like `window.api.setTheme('solarized')`.
2. **Main / preload**: the IPC handler in `src/main/index.ts` does the
   side effect (validation, writing to disk, etc.) and **dispatches a
   typed event** through the store: `store.dispatch({type: 'settings/themeChanged', payload: 'solarized'})`.
3. **Main / store**: `src/main/store.ts` runs the dispatched event
   through the shared `rootReducer`, updates its in-memory `AppState`,
   bumps a monotonic `seq`, and notifies subscribers.
4. **Main / transport**: `src/main/transport-electron.ts` subscribes to
   the store and forwards every event over the `state:event` IPC channel
   to all `BrowserWindow`s.
5. **Renderer / client store**: `src/renderer/store.ts` listens on
   `state:event`, applies the **same** shared `rootReducer` to its local
   mirror, and bumps its own seq.
6. **Renderer / hooks**: `useSyncExternalStore` notifies any component
   reading via `useSettings()` (or the slice-specific hook). React
   re-renders.

The key property: **main and renderer apply the exact same reducer
function** from `src/shared/state/`, so they're guaranteed to stay in
sync with no glue code. The renderer's "client store" is just a passive
mirror.

### Where each kind of state lives

| Kind | Lives in | Why |
|---|---|---|
| Worktree list, panes, terminal status, PR status, settings, hooks consent, onboarding quest, updater status, repo configs, lastActive timestamps | **Main store slices** | Shared world state — every viewer of this workspace needs the same value |
| `activeWorktreeId`, `activePaneId`, modal visibility (`showSettings` etc.), sidebar widths, tree expansion (`collapsedGroups`), form drafts | **Renderer `useState`** | Per-client UI focus / layout — different viewers can validly differ |
| `hooksChecked` Sets, `prevStatusesRef`, debounce refs | **Renderer `useRef`** | Per-session bookkeeping that doesn't survive reload |
| Background polling clocks, dedup state | **Main FSM/poller classes** | Lives wherever the loop runs (PRPoller, ActivityDeriver) |

The test for "is this slice or renderer state": **would a second client
connecting to the same workspace want to see the same value?** If yes,
slice. If no, renderer.

### Key files

```
src/
├── shared/state/                  # Slices imported by BOTH main and renderer
│   ├── index.ts                   # Root reducer + AppState + StateEvent union
│   ├── settings.ts                # Theme, hotkeys, claudeCommand, fonts, …
│   ├── prs.ts                     # byPath PRStatus, mergedByPath, loading
│   ├── worktrees.ts               # list, repoRoots, pending FSM entries
│   ├── terminals.ts               # statuses, pendingTools, shellActivity, panes, lastActive

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ness-dev/ness](https://github.com/ness-dev/ness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
