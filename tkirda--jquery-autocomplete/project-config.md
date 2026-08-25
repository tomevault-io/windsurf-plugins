---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project shape

TypeScript source under `src/` (~700 lines split into ~8 modules) compiles to a UMD bundle + an ESM bundle + a minified UMD + bundled `.d.ts` types in `dist/`. Edits belong in `src/*.ts`; never touch `dist/*` (the build overwrites it). The plugin is jQuery-only — `jQuery >=3.0` is a peer dependency.

`src/` layout:

| File | Role |
|---|---|
| `src/index.ts` | ESM entry point. Imports jquery, calls `installAutocomplete(jQuery)`, re-exports `Autocomplete` and types. |
| `src/umd-body.ts` | UMD entry point. References `$` as a free variable that the hand-written UMD wrapper (in `scripts/build.mjs`) provides as a factory parameter. |
| `src/jquery-plugin.ts` | `installAutocomplete($)` — registers `$.Autocomplete`, `$.fn.devbridgeAutocomplete`, and conditionally `$.fn.autocomplete` (jQuery UI guard). |
| `src/Autocomplete.ts` | The plugin class. |
| `src/defaults.ts` | The `Autocomplete.defaults` options object. |
| `src/format.ts` | Default `formatResult`, `formatGroup`, `lookupFilter`, `transformResult`. |
| `src/utils.ts` | `escapeRegExChars`, `createNode`, `keys` constants. |
| `src/jquery-ref.ts` | `export let $: JQueryStatic` set at install time via `setJQuery`. Live ES-module binding — every importer sees the value once `installAutocomplete` has run. |
| `src/types.ts` | Public types (`AutocompleteOptions`, `ResolvedOptions`, `Suggestion`, callback signatures). |

## Commands

- `npm test` — Vitest run (headless, jsdom). Single-shot, exits nonzero on failure.
- `npm run test:watch` — Vitest watch mode.
- `npm run lint` — ESLint over `test/` and `scripts/build.mjs`. **TS source is not linted by ESLint** — `tsc --noEmit` covers it via the `typecheck` script.
- `npm run typecheck` — `tsc --noEmit`. Strict mode; runs on `src/`.
- `npm run format` — Prettier rewrite of `src/`, `test/`, and `scripts/build.mjs` (100-col, 4-space, ES5 trailing commas). Demo files under `docs/` are intentionally excluded.
- `npm run format:check` — Prettier check-only, same scope. CI gate.
- `npm run build` — runs `scripts/build.mjs` (Node ESM): esbuild emits `dist/jquery.autocomplete.esm.js` (ESM) and `dist/jquery.autocomplete.js` / `.min.js` (UMD, hand-wrapped); `tsc --declaration` emits the `.d.ts` files; the version field in `devbridge-autocomplete.jquery.json` is synced from `package.json`.

## CI

`.github/workflows/ci.yml` runs on every push to `master` and every pull request: `npm ci`, then `lint`, `format:check`, `typecheck`, `test`, `build` — in that order, all required. Node 20 LTS, Ubuntu, single job. The `engines.node` field in `package.json` mirrors the runner version.

## Tests

Vitest + jsdom, headless. Specs live in `test/autocomplete.test.js`. `test/setup.js` attaches a single jQuery instance to `globalThis` / the jsdom `window`, registers `jquery-mockjax`, silences mockjax's per-request console logging, then calls `installAutocomplete(jQuery)` directly (bypassing the UMD wrapper). All test code shares one jQuery instance, one DOM, one set of plugin registrations.

`vitest.config.js` pins `pool: "forks"` with `isolate: false`. **Don't change either.** `threads` pool starved the worker handshake once we moved to TS source (esbuild transform overhead pushed startup past the 60s timeout). `isolate: false` keeps every spec in one process — same shared-module-state model the original Jasmine runner used, so describe blocks that mutate global jQuery state stay consistent.

To run a single test: `npx vitest run -t "test name substring"` or temporarily `describe.only` / `it.only`.

The demo page `docs/index.htm` is the manual test surface (Ajax lookup, local lookup with grouping, custom container, dynamic width) **and** the live demo published at https://tkirda.github.io/jQuery-Autocomplete/ via GitHub Pages (configured to serve from `master/docs`). It loads jQuery, mockjax, and the plugin itself from CDN (`cdn.jsdelivr.net/npm/devbridge-autocomplete@2/...`); open in a browser.

## Build internals

`scripts/build.mjs` does three things in order:

1. **ESM bundle** (`dist/jquery.autocomplete.esm.js`) — esbuild bundles `src/index.ts` with `external: ['jquery']`. Consumers `import 'devbridge-autocomplete'` and the plugin self-registers.
2. **UMD bundles** (`dist/jquery.autocomplete.js` and `.min.js`) — esbuild bundles `src/umd-body.ts` as IIFE (no `external`); the result is wrapped by a hand-written UMD detection shim (AMD / CommonJS / browser-global), with `$` flowing in as the factory parameter. The shim format intentionally matches the JS source that shipped before 2.0.0 so consumers don't see a contract change.
3. **Types** (`dist/*.d.ts`) — `tsc --declaration --emitDeclarationOnly`. One `.d.ts` per source file; `package.json` `types` points at `dist/index.d.ts`.

The minified UMD is ~13 KB; the unminified is ~26 KB.

## Release/version flow

1. Bump `version` in `package.json`.
2. `npm run build` — propagates the new version into the banner of each `dist/` JS file (via the build script) and syncs `devbridge-autocomplete.jquery.json`.

## Architecture notes that aren't obvious from a glance


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tkirda/jQuery-Autocomplete](https://github.com/tkirda/jQuery-Autocomplete) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
