---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run build              # esbuild → dist/iife/no.js (minified + sourcemaps)
npm test                   # Jest (jsdom)
npm run test:watch         # Jest watch mode
npm run test:coverage      # Jest with coverage
npm run test:e2e           # Playwright (chromium, firefox, webkit)
npm run test:e2e:headed    # Playwright with visible browser
npm run test:all           # Jest + Playwright
npm run bench              # Loop performance benchmarks (__benchmarks__/)
npm start                  # Dev server for docs site (port 3000)
```

Run a single test file: `npx jest --no-coverage __tests__/filters.test.js`

Coverage for a specific module: `npx jest --coverage --collectCoverageFrom='src/<module>.js' __tests__/<module>.test.js`

No linter configured.

## Architecture

**No.JS** is an HTML-first reactive framework — one `<script>` tag, zero external dependencies, no build step for users. Walks the DOM on init, matches HTML attributes to directives, executes them by priority.

### Build outputs

| Format | Entry | Output | Use case |
|--------|-------|--------|----------|
| IIFE | `src/cdn.js` | `dist/iife/no.js` | CDN `<script>` tag |

`build.js` uses esbuild.

### Core modules (src/)

- **index.js** — Public API, plugin system, lifecycle. Exposes `NoJS.config()`, `init()`, `use()`, `directive()`, `filter()`, `validator()`, `router`, `store`, etc.
- **globals.js** — Shared mutable state: `_config`, `_stores`, `_interceptors`, plugin registry. `_log()` only fires when `_config.debug` is true; `_warn()` always fires.
- **context.js** — Reactive Proxy-based contexts with parent chain inheritance (lexical scoping). `createContext()`, `findContext()`, `$watch`. Batch operations via `_startBatch()` / `_endBatch()`.
- **evaluate.js** — CSP-safe expression parser (no eval/Function). Allow-list approach: `_SAFE_GLOBALS` for JS builtins, `_BROWSER_GLOBALS` for curated browser APIs. Expression cache is LRU-bounded (`_config.exprCacheSize`, default 500). Functions: `evaluate()`, `resolve()`, `_interpolate()`.
- **dom.js** — DOM walking, template loading (`_loadTemplateElement`), disposal (`_disposeTree`), `processTree()`. Templates with `route` attribute are treated as route templates; without it, they're content-includes that get injected inline.
- **router.js** — SPA routing: hash/history mode, file-based routes, nested outlets, View Transition API, guards, prefetch, head management. `_loadNestedIndexRoutes()` handles `route-index` on nested outlets after VT completes.
- **fetch.js** — HTTP with interceptors, caching, retries, CSRF, credential handling. Sentinel symbols: `CANCEL`, `RESPOND`, `REPLACE`.
- **registry.js** — Directive registration via `registerDirective(name, handler)`. Core directives frozen after init — plugins can add but not override.
- **filters.js** — 32+ built-in filters (currency, date, uppercase, etc.). Self-register on import. Custom: `NoJS.filter('name', fn)`.
- **directives/** — 15+ directive files organized by category (state, http, binding, conditionals, loops, styling, events, refs, validation, i18n, dnd, head, animations). One file per category, side-effect imports for registration.
  - **Loop/else pattern:** loop directives (`foreach`, `each`, `for`) support empty-state rendering via `else="templateId"` on the loop element, referencing a `<template>`. The sibling else pattern was removed in v1.15. The conditional `else` handler skips elements that carry a loop directive.

### Directive priority order

0 (state/store) → 1 (fetch/i18n/head) → 2 (computed/watch) → 5 (ref) → 10 (structural: if/each/for/use) → 15 (dnd) → 20 (bind/events/style/model) → 30 (validate)

### Code style

- Private API uses `_` prefix: `_config`, `_loadRemoteTemplates()`, `_disposeTree()`
- Logging: `_log()` / `_warn()` from `globals.js` — never `console.log`
- Global state: always import from `globals.js`
- Caching: `Map` objects (`_templateHtmlCache`, `_i18nCache`, `_autoTemplateCache`)

## Mandatory Safety Rules

These rules exist because real bugs were found and fixed. Every rule has a tracked origin issue.

### 1. Disposal before clearing DOM

Always `_disposeTree()` children BEFORE `innerHTML = ""`. Iterate `el.children`, not the parent (disposing the parent breaks re-rendering).

```js
// WRONG — leaks contexts, listeners, watchers
el.innerHTML = "";

// RIGHT
for (const child of [...el.children]) _disposeTree(child);
el.innerHTML = "";
```

Applies to: every directive that swaps content (`if`/`else`, `each`/`foreach`, `get`/`post`, `switch`/`case`, `use`, `error-boundary`).

### 2. Event listener cleanup

Always register cleanup via `_onDispose()` immediately after `addEventListener`. Exception: `{ once: true }` auto-removes.

```js
el.addEventListener(event, handler, opts);
_onDispose(() => el.removeEventListener(event, handler, opts));
```

### 3. Watcher unsubscribe

Always capture `$watch()` return value and register via `_onDispose()`.

```js
const unwatch = ctx.$watch(fn);
_onDispose(() => { if (unwatch) unwatch(); _storeWatchers.delete(fn); });
```

### 4. Timer guards


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [no-js-dev/nojs](https://github.com/no-js-dev/nojs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
