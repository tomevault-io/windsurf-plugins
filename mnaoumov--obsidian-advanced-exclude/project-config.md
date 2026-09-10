---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Obsidian plugin that enhances Obsidian's `Files and links > Excluded files` setting with full `gitignore` syntax. Ignored files become invisible to Obsidian (Files pane, Backlinks, etc.), not just dimmed.

Built on `obsidian-dev-utils`. Patches Obsidian's `FileSystemAdapter` / `CapacitorAdapter` and `FileExplorerView` to filter ignored paths at the source.

## Commands

| Task              | Command                    |
|-------------------|----------------------------|
| TypeScript check  | `npm run build:compile`    |
| Build             | `npm run build`            |
| Dev (watch)       | `npm run dev`              |
| Lint              | `npm run lint`             |
| Lint (fix)        | `npm run lint:fix`         |
| Format            | `npm run format`           |
| Format (check)    | `npm run format:check`     |
| Spellcheck        | `npm run spellcheck`       |
| Markdown lint     | `npm run lint:md`          |
| Markdown lint fix | `npm run lint:md:fix`      |
| Unit tests        | `npm test`                 |
| Coverage          | `npm run test:coverage`    |
| Integration tests | `npm run test:integration` |
| Commit (wizard)   | `npm run commit`           |

## Architecture

- **Root config files** are thin re-exports — actual logic lives in `scripts/`:
  - `eslint.config.mts` → `scripts/eslint-config.ts`
  - `commitlint.config.ts` → `scripts/commitlint-config.ts`
  - `vitest.config.ts` → `scripts/vitest-config.ts`
  - `.markdownlint-cli2.mjs` → `scripts/markdownlint-cli2-config.ts` (via jiti)
  - `.nano-staged.mjs` → `scripts/nano-staged-config.ts` (via jiti)
- **`scripts/`** — all npm script entry points (`jiti scripts/<name>.ts`)
- **`src/`** — plugin source:
  - `main.ts` — Obsidian entry point (default export of `Plugin`)
  - `plugin.ts` — `Plugin` class, wires up child components
  - `ignore-patterns-component.ts` — owns gitignore matching, IndexedDB cache, `.obsidianignore` / `.gitignore` reads
  - `vault-model.ts` — `VaultModel`: the in-memory shadow tree and its bottom-up visibility (`recomputeAll`, `applyDelta`, `seedHidden`)
  - `index-projection-component.ts` — `IndexProjectionComponent`: projects the model onto Obsidian's index in one event-free pass, drives the explorer, owns fast-enable and apply progress
  - `manual-index-hider.ts` — `ManualIndexHider`: S6 direct index mutation — hides/re-inserts files with no events, keeps the in-memory snapshots a same-session un-ignore restores from
  - `vault-path-store.ts` — IndexedDB persistence of the hidden set + universe signature (`IndexedDatabaseVaultPathStore`)
  - `universe-signature.ts` — order-independent signature of the file universe; a mismatch vetoes fast-enable
  - `file-tree-component.ts` — drives Files pane add/delete based on ignore state
  - `data-adapter-safe.ts` — read/write/stat wrappers that survive missing files
  - `indexed-database-utils.ts` — IndexedDB request → promise helper
  - `constants.ts` — shared constants (`.gitignore` / `.obsidianignore` names, root path)
  - `publish-compatibility-warning-component.ts` — warns when Obsidian Publish is on while `excludeMode === Full`
  - `restore-notice-component.ts` — restores hidden files synchronously on unload (added last so it unloads first)
  - `update-progress-notice-component.ts` — progress notice shown while a projection is applied
  - `plugin-settings*.ts` — settings model, component, settings tab
  - `patches/` — monkey-patches on Obsidian internals:
    - `adapter-patch-component.ts` — dispatches to file-system or capacitor variant
    - `adapter-patch-base-component.ts` — shared `MonkeyAroundComponent` base for both adapter patches
    - `file-system-adapter-patch-component.ts`, `capacitor-adapter-patch-component.ts` — patch `reconcileFile{Creation,Internal}`
    - `vault-load-patch-component.ts` — intercepts initial vault load
    - `file-explorer-view-on-create-patch-component.ts` — patches `FileExplorerView.onCreate`
- **Test files** live next to the source: `alpha.ts` → `alpha.test.ts`. Integration tests are named by the vitest project that runs them: `.desktop.` / `.android.` / `.cross-platform.` / `.no-app.` / `.desktop-performance.` / `.demo-vault.` / `.desktop-capture.` / `.android-capture.` + `.integration.test.ts`.
- **`main` field** points to `src/main.ts` (Obsidian plugin source entry — built artifact is `dist/build/main.js`, not published to npm).

## Conventions

- **Mocking**: prefer `strictProxy<T>({...})` / `StrictProxyPartial<T>` from `obsidian-dev-utils/strict-proxy` over `as unknown as T` and over `Record<string, unknown>` mock interfaces. Strict proxies throw on uninitialized property access, so mistakes surface in the failing test instead of silently passing.
- **Constructor params**: components take a single `{...}Params` object. When mocking a component constructor in tests, type the params with the real exported `...ConstructorParams` interface — export the interface from the source file if needed.
- **v8 ignore**: only block form (`/* v8 ignore start -- reason. */` … `/* v8 ignore stop */`) is honored. Single-line `/* v8 ignore next */` does not work.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mnaoumov/obsidian-advanced-exclude](https://github.com/mnaoumov/obsidian-advanced-exclude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
