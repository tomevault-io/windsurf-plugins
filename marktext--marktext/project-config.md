---
trigger: always_on
description: This file provides guidance to Claude Code when working inside `packages/muya`.
---

# CLAUDE.md (packages/muya)

This file provides guidance to Claude Code when working inside `packages/muya`.

> **Location.** `packages/muya` is the TypeScript rewrite of muya (upstream: <https://github.com/marktext/muya>), migrated into this marktext monorepo and published as `@muyajs/core`. The desktop renderer now consumes `@muyajs/core` as its editor engine; the legacy JS engine `packages/muyajs` (`@marktext/muyajs`, the `muya/` alias) is being retired and only a handful of call sites still reference it. `packages/muya` keeps its own toolchain (ESLint/antfu, stylelint, madge, vitest), and the marktext-root ESLint ignores `packages/muya/**` — treat it as a self-contained package with its own conventions.

## Layout inside `packages/muya`

- `src/` — `@muyajs/core` TypeScript source. Public API entrypoint is `src/index.ts`.
- `test/spec/` — CommonMark / GFM conformance suites (run via `test:spec`, separate vitest config).
- `examples/` — `muya-examples`, a Vite vanilla-TS demo that consumes `@muyajs/core` via `workspace:*`. Listed as its own workspace in the repo-root `pnpm-workspace.yaml`.
- `e2e/` — `muya-e2e`, Playwright real-browser E2E suite. Self-contained host page under `e2e/host/`. See `e2e/README.md` and `e2e/BACKLOG.md`.
- `eslint.config.mjs`, `.stylelintrc`, `.madgerc` — package-local tooling. The marktext-root ESLint explicitly ignores `packages/muya/**`, so muya self-lints with its own antfu-based config.

Stub packages (`packages/facade`, `packages/findReplace`) from the upstream muya monorepo were not migrated — they had no source.

## Commands

Run from the marktext repo root.

- `pnpm -C packages/muya/examples dev:demo` — start the examples Vite dev server. (Upstream `pnpm dev` / Turbo `dev:demo` is not wired here — run vite directly.)
- `pnpm -C packages/muya build` — `tsc && vite build`, emits `lib/{es,umd,cjs}` and `lib/types`.
- `pnpm -C packages/muya test` / `pnpm -C packages/muya coverage` — Vitest unit tests (co-located under `src/**/__tests__/`). Single file: `pnpm -C packages/muya exec vitest run path/to/file.test.ts`.
- `pnpm -C packages/muya test:spec` — CommonMark 0.31 + GFM 0.29-gfm fixture suites against `renderToStaticHTML(..., { sanitize: false })`. `test:spec:commonmark` / `test:spec:gfm` scope to one suite. Pass/fail counts are locked by `test/spec/expected-failures.json`: any listed example that starts passing fails the suite (remove it from the list); any unlisted example that starts failing fails the suite. Compliance can only go up. Baseline lives in `test/spec/conformance.md` (CommonMark 87.7% / GFM 86.3% at PR-6a).
- `pnpm -C packages/muya lint` / `pnpm -C packages/muya lint:fix` — ESLint over `src test` (antfu config; rules below).
- `pnpm -C packages/muya lint:types` — `tsc --noEmit`.
- `pnpm -C packages/muya lint:css` — Stylelint over `src/**/*.css`.
- `pnpm -C packages/muya check-circular` — `madge --circular src/index.ts`. CI enforces this.
- `pnpm -C packages/muya/e2e e2e` — Playwright E2E (chromium/firefox/webkit). `e2e:install` is a one-time browser install. CI (`muya-e2e.yml`) runs Chromium only; Firefox + WebKit are configured in `playwright.config.ts` and runnable locally, but excluded from the CI matrix until the engine-independent rewrites in BACKLOG Phase 3 land (triple-click selection, search-replace mutation timing).

Engines: Node ≥20.19 (matches marktext root). Build target is `chrome70`.

## Architecture

### Entry point and plugin system

`src/muya.ts` exports the `Muya` class. UI plugins are registered globally via the static `Muya.use(Plugin, options)` and instantiated inside `muya.init()`. Plugins are keyed by `Plugin.pluginName` and stored on `muya._uiPlugins`. The plugin set in `examples/src/main.ts` is the canonical reference for wiring up toolbars, selectors, and menus.

`new Muya(element, options)` replaces the passed-in element with a new `contenteditable` div (`getContainer` in `muya.ts`), then constructs `EventCenter`, `Editor`, `Ui`, and `I18n`. Nothing renders until `muya.init()` runs `Editor.init()`, which calls `registerBlocks()` and creates the root `ScrollPage`.

### The `Editor` (`src/editor/index.ts`)

Holds the runtime modules: `JSONState`, `InlineRenderer`, `Selection`, `Search`, `Clipboard`, `History`, and the root `ScrollPage`. It owns `activeContentBlock` (the focused leaf) and routes DOM events (`click`, `input`, `keydown`, `keyup`, `compositionstart/end`) merged via RxJS to the active block's handlers (`clickHandler`, `inputHandler`, etc.). Anything that listens to user input on a block ultimately flows through this dispatch.

`Editor.updateContents(operations, selection, source)` applies `ot-json1` operations to the live block tree. The `pick`/`drop` walk is hand-rolled from `ot-json1.apply` so it can call `block.replaceWith`, `container.insertBefore`, `ScrollPage.loadBlock(name).create(...)`, and `otText.type.apply` on the matching subdocument — the block tree and the JSON state stay in lockstep.

### Block tree


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marktext/marktext](https://github.com/marktext/marktext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
