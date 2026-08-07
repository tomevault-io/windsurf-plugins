---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Gitty is an Electron desktop git history browser with four panes: working tree
(top left), diff (top right), commit log (bottom left) and an interactive shell
(bottom right). See README.md for the user-facing behaviour of each pane.

**Everything user-visible is English** — the interface, README, CHANGELOG and
commit messages. Conversation with the user may be in another language, but
nothing that lands in the repository is.

## Commands

```bash
./run.sh [repo]          # build if stale, launch detached, print the pid
./run.sh --fg [repo]     # same but attached to the terminal
./run.sh --dev [repo]    # electron-vite dev with hot reload
npm run typecheck        # tsc over both tsconfigs — the only automated check
npm run build            # electron-vite build into out/
./setup.sh               # symlink run.sh as `gitty` into ~/.local/bin
```

A detached run writes everything to
`${XDG_STATE_HOME:-~/.local/state}/gitty/gitty.log`; use `--fg` when you want
the output inline.

There is **no test suite and no linter configured**. `npm run typecheck` is the
whole automated safety net; run it after every change. Verification beyond that
is visual — see below.

`npm install` runs `electron-rebuild -f -w node-pty` via `postinstall`; node-pty
is native and must match Electron's ABI, not Node's. If the terminal pane fails
to start after changing Electron versions, re-run that rebuild.

## Verifying changes visually

The UI cannot be checked by reading code, and this is a Wayland session where
X11 screenshot tools capture nothing. Screenshot it from inside Electron
instead: build, patch a `capturePage` call into `out/main/index.js` (the built
bundle, never the source) at the `ready-to-show` handler, run with
`GITTY_REPO=<repo>` and an env var carrying a delay plus an output path, then
read the PNG. Drive the UI first by passing a snippet to
`win.webContents.executeJavaScript` — clicking `.commit-row`, `.row` or header
buttons, or dispatching a `contextmenu` MouseEvent. `out/` is gitignored and
rebuilt, so the patch is throwaway.

Use a repository with real history for this; this repo's own log is short.

## Architecture

Three processes with a hard boundary between them:

- **`src/main/`** — owns everything privileged: git subprocesses, the pty, the
  fs watcher, dialogs, clipboard, `shell.openPath`.
- **`src/preload/`** — the only bridge. Exposes a frozen `window.gitty` API over
  `contextBridge`; the renderer has no node integration and no direct IPC.
- **`src/renderer/`** — React UI, pure presentation over that API.
- **`src/shared/types.ts`** — the contract both sides import.

Adding a capability means touching all three: an `ipcMain.handle` in
`src/main/index.ts`, a method in `src/preload/index.ts`, and a type in
`src/shared/types.ts`.

### Git access

`src/main/git.ts` shells out to `git` via `execFile` — no git library. Parsing
relies on NUL-separated machine formats (`status --porcelain=v2 -z`,
`--name-status -z`, `ls-tree -z`) so paths with spaces and renames survive;
rename records carry an extra NUL field, which is why those loops advance the
index by hand. Diffs above 2 MB are truncated with a notice rather than sent
whole. Whatever `git` is on `PATH` is what the app shows.

### Multiple repositories, tabs

`App.tsx` is a thin tab manager: the list of open roots, which is active, the
app-wide preferences (theme, font size, wrap, …), and the settings dialog. Each
open root renders one `RepoTab` (`src/renderer/src/RepoTab.tsx`) owning that
repository's whole session — status, log, the `View`, selected file, context
menu, and its own `TerminalsPane`. Inactive tabs stay mounted (`display: none`),
so switching never disturbs another repo's view state or shells. The main
process keeps one watcher per root and tags `repo:changed` with the root, so
each tab refreshes only its own repository. The tab bar (basename, dirty dot,
close button, `+` to open) sits below the panes, with an empty state when every
tab is closed. `react-resizable-panels` keeps layout state per Group id, so
`RepoTab` suffixes its ids with the root.

Two things the tab shells must not lose. **`min-width: 0`** on `.tab-content`,
`.repo-tab-shell` and `.repo-tab`: a flex item defaults to `min-width: auto`, so
without it a tab is stretched by its own nowrap content — long paths, long
commit subjects — and its panel group ends up wider than the window. The panel
percentages stay correct while the total is wrong, which shows up as one pane
squeezed to a sliver (its header buttons clipped away) and another pushed off
screen entirely. And **`disabled={!active}`** on every Group of a hidden tab:
the library hit-tests the pointer against every registered group, and a
`display: none` group reports a zero-sized rect.

### Hiding panes

`src/renderer/src/panes.ts` holds the `PaneVisibility` record; `App.tsx` owns it
(app-wide, like the other view preferences, persisted under `gitty.panes`) and
`RepoTab` simply does not render a hidden pane's `Panel` — nor the `Separator`
beside it, nor a whole row when both its panes are gone. Two consequences worth

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [baojie/gitty](https://github.com/baojie/gitty) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
