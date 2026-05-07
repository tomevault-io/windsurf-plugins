---
trigger: always_on
description: Client-side, high-performance network waterfall library (WebPageTest-style).
---

# Waterfall Tools: AI Agent Guidance

Client-side, high-performance network waterfall library (WebPageTest-style).

## Workflow (mandatory)

- **Start:** read `Docs/Architecture.md` and `Docs/Plan.md`.
- **End:** update `README.md`, `Docs/Plan.md`, `Docs/Architecture.md`, and `AGENTS.md` (this file is long-term project memory).
- **Hygiene:** delete any throw-away diagnostic scripts, `.log` files, and scratch outputs from the repo root before concluding.
- **API changes** to public signatures (`WaterfallTools` methods, `renderTo()` options) MUST be reflected in `README.md` and noted here.

## Core architecture

- Vanilla JS only in library core. No React/Vue/Svelte/Angular. External libs allowed if they improve the architecture.
- Use `<canvas>` for waterfalls — no per-request DOM elements (1000+ requests are common).
- Every input format maps to **Extended HAR** before rendering or output. HAR `log.creator` must identify `waterfall-tools`. Custom fields prefix with `_` (e.g. `_priority`, `_load_ms`, `_ttfb_ms`, `_bytesIn`). Schema: `Docs/Extended-HAR-Schema.md`. Types: JSDoc in `src/core/har-types.js`.
- Pluggable, decoupled modules. Orchestrator transports verified HAR across renderers/outputs without implicit mutation. Tree-shakeable exports.
- Target: latest stable Chrome, Firefox, Safari, Node. No polyfills or transpilation. UMD abandoned — ESM only.
- Licenses: MIT / BSD / Apache-2 / ISC / MPL only. **No GPL** in any form.

## File layout / isomorphism

- `src/core/` and top-level `src/inputs/*.js` — strictly isomorphic (Node + browser).
- `src/inputs/cli/[format].js` — CLI wrappers (Node-only, keeps bundlers away from `process.*`).
- `src/inputs/utilities/[format]/` — format-specific decoders.
- `src/platforms/browser/`, `src/renderer/`, `src/embed/` — Web APIs only.
- `src/platforms/node/` — Node APIs only (`fs`, `zlib`, `crypto` dynamic-imported from isomorphic code).
- Structure must allow dropping in WASM for heavy future work (trace decompress, frame unpacking).

## Build

- Rollup, not Rolldown. Outputs: `waterfall-[hash].js`, `tcpdump-[hash].js`, `decompress-[hash].js`, plus a 41-byte `waterfall-tools.es.js` stub that imports the hashed payload (enables 1-year max-age caching).
- Static artifacts have Brotli `.br` counterparts, level 11.
- Orchestration: `scripts/build.js`.
- Viewer uses `<script type="importmap">` pointing `waterfall-tools` → `dist/browser/waterfall-tools/waterfall-tools.es.js`. `vite.dev.config.js` aliases the bare specifier to `src/core/waterfall-tools.js` for HMR (`npm run dev:viewer`).

## Linting

- ESLint flat config at `eslint.config.js`. Run with `npm run lint` (or `npm run lint:fix`). `npm run build` runs lint first and aborts on any lint output — the lint step uses `--max-warnings 0`, so warnings are treated as errors.
- **Only project code is linted**, never dependencies or third-party bundles. Ignored paths: `node_modules/`, `dist/`, `bin/demo/` (vite build output), `Sample/`, `src/viewer/public/netlog-viewer/` (vendored Chrome NetLog viewer), `coverage/`, `**/*.min.js`. When adding new third-party code drops, extend the `ignores` entry in `eslint.config.js` to match.
- Per-directory language options partition the project by runtime: browser-only (renderer, viewer, demo, embed, browser platform), Node-only (CLI wrappers, bin, scripts, vite/vitest configs, node platform), isomorphic (core, top-level inputs, input utilities, outputs — gets both browser + node globals), service worker (`src/viewer/public/sw.js`), and Cloudflare Worker (`cloudflare-worker/`). Keep these globs in sync with `src/` layout when adding new directories.
- **All new first-party code must be covered by lint.** When you add a new file, directory, or top-level path under our control (a new `src/` subtree, a new `scripts/` directory, a new `bin/` entry, a new Worker, etc.), verify it is matched by one of the `files` globs in `eslint.config.js` and run `npm run lint` to confirm it is picked up. If it isn't matched, extend the nearest appropriate `files` entry (or add a new language-options block if the runtime differs) so the file is linted — never leave new first-party code outside lint coverage. Only vendored third-party drops belong in `ignores`.
- **Fix the underlying issue, don't silence the rule.** Any lint warning or error must be resolved by correcting the code. Only in extreme cases — where the rule is genuinely inapplicable to the specific construct (e.g. an intentional `catch(e)` with no body, a deliberately-unused placeholder parameter needed for API shape) — use a scoped `// eslint-disable-next-line <rule>` with an inline comment explaining why. Never disable rules file-wide or in the config to work around real bugs.
- Unused-var escape hatch: prefix with `_` (the config allows `^_` for args, vars, and caught errors) rather than disabling the rule.
- **CI gate:** `.github/workflows/ci.yml` runs `npm ci` + `npm run lint` + `npm run build` on Node 22 for every PR targeting `main`. Lint warnings or build failures block the merge. Keep the workflow's Node version in step with `scripts/build.js` expectations when bumping engines, and update the job steps when build commands change (e.g. new `npm run test`-level gate).

## Testing


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pmeenan/waterfall-tools](https://github.com/pmeenan/waterfall-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
