---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Branching workflow

Integration-branch model (simplified git-flow):

- `main` — release branch. Always stable. Only receives merges from `develop`, **squashed**, after the extension is tested in a browser. Store builds are cut from here.
- `develop` — integration branch. Feature branches merge here (regular merge, history preserved) and are tested together.
- `feature/NNN-name` — one per task, branched from `develop`, named after the task it implements. Merged into `develop`, then deleted.

Per-task flow: `git switch develop` → `git switch -c feature/NNN-name` → work → merge into `develop` → test in browser → (when `develop` is stable) squash-merge `develop` into `main`. Never commit directly to `main`. Commit/push only when the user asks.

## What this is

A WebExtension that does two-way bookmark sync between the browser (Firefox MV2 / Chrome MV3) and [Raindrop.io](https://raindrop.io). TypeScript, bundled with webpack, no test suite. Cross-browser support is achieved with `webextension-polyfill` — all code imports `browser` from it (never `chrome.*`).

## Commands

```bash
npm install                  # Install dependencies
npm run watch:firefox        # Dev build + rebuild on change (also watch:chrome)
npm run build:firefox        # Production build → dist/firefox (also build:chrome, build:all)
npm run start:firefox        # Launch browser with the built extension via web-ext
npm run lint                 # web-ext lint against dist/firefox (build first)
npm run package:all          # Build + package both browsers for store submission
```

There is no test runner and no standalone `tsc` typecheck script — type errors surface only through `ts-loader` during a webpack build. After changing code, run `npm run build:firefox` (or `:chrome`) to verify it compiles. `lint` requires `dist/firefox` to already exist.

To load manually: Firefox → `about:debugging` → Load Temporary Add-on → `dist/firefox/manifest.json`; Chrome → `chrome://extensions` → Load unpacked → `dist/chrome/`.

## Build system

`webpack.config.js` takes `--env browser=firefox|chrome` and emits to `dist/<browser>/`. It compiles three entry points (`background`, `popup`, `options`), copies the matching `manifests/<browser>.json` to `manifest.json`, and copies icons + HTML/CSS. `optimization.minimize` is **off** on purpose (store reviewers need readable source). The two manifests are the only browser-specific files: Firefox is MV2 (persistent background page, `browser_action`, `host_permissions` folded into `permissions`); Chrome is MV3 (service worker, `action`, separate `host_permissions`).

## Architecture

All real logic lives in the background script (`src/background/`). The popup and options pages are thin UIs that talk to it exclusively via `browser.runtime.sendMessage`. The single message switch in `src/background/index.ts:handleMessage` is the entry point for every UI action — to add a feature, add a `case` there plus a matching action string in `src/types/messages.ts`.

Both directions flow through **one three-way reconcile** (`syncManager.ts:reconcileAllMappings`), driven by two triggers:

- **Real-time:** `bookmarkListeners.ts` listens to `bookmarks.onCreated/onRemoved/onChanged/onMoved`. Each handler runs cheap relevance guards and, if relevant, arms an 800 ms trailing-debounce timer (`scheduleReconcile`) that dynamically `import('./syncManager')` and calls `reconcileAllMappings()`. Events carry **no payload** — reconcile re-derives everything from the diff. A burst (bulk import, multi-drag) collapses into one pass.
- **Periodic:** the `raindrop-sync-interval` alarm (configurable 1–60 min) calls the same `reconcileAllMappings()`, catching up either direction — including anything a real-time trigger missed (e.g. the MV3 SW died before the debounce fired).

**`reconcileAllMappings` is the core.** For every `BookmarkLink` (the baseline) it compares the browser side and the Raindrop side against the last-synced `contentHash`/`mappingId` and derives a direction via the pure decision module `reconcile.ts` (`decideBookmarkAction`): push / pull / delete-in-raindrop / delete-in-browser / drop-link; both-changed → Raindrop wins. No baseline → union (create/adopt by URL, **never delete**). It also owns nested-folder mirroring (`reconcileFolderTree` / `syncFolderWithChildren`, creates Raindrop collections to match folder trees, depth-capped at `MAX_SYNC_DEPTH`), bidirectional folder rename (`decideRenameAction`), and full re-sync (one global union pass). It holds a stale-tolerant `reconcile_lock` in `storage.local` (5-min timeout) so only one pass runs at a time across SW restarts.

### Loop prevention (the central design problem)

Sync writes bookmarks, which fire bookmark events, which would trigger more reconciles — an infinite loop. Defenses, all of which must be preserved when editing sync code:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [viktorkasap/raindrop-bookmark-sync](https://github.com/viktorkasap/raindrop-bookmark-sync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
