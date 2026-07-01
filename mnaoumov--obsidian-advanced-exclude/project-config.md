---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

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
  - `file-tree-component.ts` — drives Files pane add/delete based on ignore state
  - `data-adapter-safe.ts` — read/write/stat wrappers that survive missing files
  - `plugin-settings*.ts` — settings model, component, settings tab
  - `patches/` — monkey-patches on Obsidian internals:
    - `adapter-patch-component.ts` — dispatches to file-system or capacitor variant
    - `file-system-adapter-patch-component.ts`, `capacitor-adapter-patch-component.ts` — patch `reconcileFile{Creation,Internal}`
    - `vault-load-patch-component.ts` — intercepts initial vault load
    - `file-explorer-view-on-create-patch-component.ts` — patches `FileExplorerView.onCreate`
- **Test files** live next to the source: `foo.ts` → `foo.test.ts`. Integration tests use suffixes `.desktop.integration.test.ts` / `.android.integration.test.ts`.
- **`main` field** points to `src/main.ts` (Obsidian plugin source entry — built artifact is `dist/build/main.js`, not published to npm).

## Conventions

- **Mocking**: prefer `strictProxy<T>({...})` / `StrictProxyPartial<T>` from `obsidian-dev-utils/strict-proxy` over `as unknown as T` and over `Record<string, unknown>` mock interfaces. Strict proxies throw on uninitialized property access, so mistakes surface in the failing test instead of silently passing.
- **Constructor params**: components take a single `{...}Params` object. When mocking a component constructor in tests, type the params with the real exported `...ConstructorParams` interface — export the interface from the source file if needed.
- **v8 ignore**: only block form (`/* v8 ignore start -- reason. */` … `/* v8 ignore stop */`) is honored. Single-line `/* v8 ignore next */` does not work.
- **Commit messages**: Conventional Commits. Use `npm run commit` (czg) for the interactive wizard.

## Current Task

**Pending dev-utils release — simplify the projection yield.** `obsidian-dev-utils`
commit `fa07bc1e` ("feat: add fallback to requestAnimationFrameAsync") moved the
rAF-vs-timeout race into `requestAnimationFrameAsync(fallbackTimeoutInMilliseconds = 100)`
itself — identical default to the plugin's local `yieldToPaint()`. It is committed in
dev-utils but **not yet published** (npm latest `80.1.0` predates it; the maintainer will
release a new version). Once a dev-utils version containing `fa07bc1e` is published and
installed: bump the plugin's `obsidian-dev-utils` dependency, then in
`src/index-projection-component.ts` delete `yieldToPaint()` and `BACKGROUND_YIELD_FALLBACK_MS`
and call `requestAnimationFrameAsync()` directly at both yield points (recompute `yieldFn`
and `reportApplyProgress`); keep the "keeps progressing when the window is hidden" test (it
still passes against the library's built-in fallback). Do NOT refactor before then — the
no-arg call against the current `80.1.0` has no fallback, so the hidden-window test would hang.

## Design & History (S6, publish-compatibility, in-memory tree rewrite)

**Publish-compatibility warning shipped.** `src/publish-compatibility-warning-component.ts`
(a `LayoutReadyComponent`, wired in `plugin.ts`) warns when Obsidian Publish is enabled

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mnaoumov/obsidian-advanced-exclude](https://github.com/mnaoumov/obsidian-advanced-exclude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
