---
trigger: always_on
description: Instructions for coding agents and humans working on yace.
---

# AGENTS.md

Instructions for coding agents and humans working on yace.

## What this is

yace is a tiny (under 2KB gzip core, zero dependencies) framework-agnostic
browser code editor: a transparent `textarea` stacked over a `pre`. The
`textarea` owns input, caret, IME, and accessibility; the `pre` renders
highlighted HTML produced by a highlighter function. Plugins transform
textarea state on each keystroke. The repo also holds a promo landing
site under `site/` (see `## site`); the library is the product, the site
is a consumer of it.

## Commands

- `npm test` — c8 coverage enforced at 100% lines / 100% functions /
  99% branches
- `npm run typecheck`
- `npm run test:e2e` — all Playwright projects (see `## site` for the
  groups)
- `npm run dev` — the landing on `http://localhost:5714`,
  live-transpiles `src/*.ts`
- `npm run build` — ends with a size gate: fails when `dist/index.js`
  gzip reaches 2000 B, so the public "under 2KB" claim stays enforced;
  `npm run size` re-runs the gate on an existing dist
- `npm run attw` — packaging check; its flags are load-bearing, see
  Gotchas
- `npm run prettier:check` — prettier 3 defaults; `test/.prettierrc`
  (tests) and the root `.prettierrc` (`*.html`) widen printWidth to 120,
  intentional per-context overrides

## Architecture in five lines

- `src/index.ts` — the editor: renders textarea + pre into the given
  root, syncs them on every input
- `src/styles.ts` — inline styles that keep the two layers pixel-aligned
- Highlighters are external and run as a pipeline: the `highlighters`
  option is an array of `(value, context?) => html`; the core bundle
  ships none (keeps it tiny), but ready-made ones live under
  `yace/highlighters/*`
- Plugins: `(props, event) => Partial<props> | void`, reduced left to
  right on keydown/input/compositionend; results merge into the textarea
  state
- `history()` must be FIRST in the plugin chain — it checkpoints state
  before other plugins transform it

## Import contract

- `import { Yace } from "yace"` — named export, core only; plugins and
  highlighters are never re-exported from the root (keeps the core
  bundle tiny)
- Plugins only via subpath: `import { tab } from "yace/plugins/tab"`
  (the named export mirrors the subpath name); `isKey` lives under
  plugins as plugin-authoring tooling
- Highlighters via subpath:
  `yace/highlighters/{code,sliceGlitch,shimmer,markdown}`. These
  are enumerated explicitly, not a wildcard, so the internal shared
  chunks (the `words` scanner, `injectStyles`, the inlined mdhl) stay
  unexported
- Barrels re-export the whole set: `import { history, tab } from
  "yace/plugins"` and `import { code, shimmer } from "yace/highlighters"`
  (the highlighters barrel also re-exports the `CodeRule` type). A
  bundler tree-shakes a pure re-export barrel down to byte-identical
  deep-import output (probed); a no-bundler consumer (CDN, import map)
  fetching a barrel pays for every sibling instead, so deep subpaths
  stay the documented path for CDN/import-map use and the e2e fixtures
  keep their deep imports on purpose. Against live `src` the
  highlighters barrel does not just cost extra — it fails to load
  without an import-map entry for `mdhl` (see `## mdhl vendoring`). The
  landing is the one deliberate exception: its plugins come through the
  `yace/plugins` barrel to mirror the README quick start, eating the
  sibling cost; its highlighters stay deep — required, see `## site`
- Barrel shaking rests on two facts, it is not free: `sideEffects: false`
  in package.json AND every re-exported module being free of top-level
  side effects — the highlighter factories inject their CSS inside the
  factory call, never at import. rollup emits a bare `import "./words.js"`
  on the highlighters facade (words is a shared internal chunk); a
  consumer drops it only because `sideEffects: false` permits it. A
  module-level side effect anywhere in the set would defeat the shake and
  pull dead siblings into every consumer bundle
- The exports map is the encapsulation boundary: deep paths like
  `yace/dist/...` are closed, internal dist layout may change freely
- Dist is ESM-only, exports are named. `require("yace")` on Node 22+
  (`require` of an ESM module) returns the module namespace, so
  `const { Yace } = require("yace")` works with no interop shim; native
  `import` is unaffected. Jest's default CJS runtime cannot load ESM —
  vitest or jest's ESM mode works. Typed CJS consumers need TS
  `module` `nodenext` (implies the matching `moduleResolution`, 5.8+):
  `node16` models older Node and rejects `require` of ESM with TS1471
  (verified against the tarball).
  Re-adding `.cjs`/`.d.cts` later is additive and non-breaking (a minor)
- After 1.0.0 the contract is frozen; evolution is additive only (new
  plugin or highlighter = new subpath = minor)

## Invariants and their WHY

- The highlighter pipeline is the XSS boundary. Stage 0 receives the raw
  value with `{ html: false }` and MUST HTML-escape it. Later stages
  receive the previous stage's HTML with `{ html: true }` and must be
  HTML-aware: copy tag runs verbatim, match only inside text, and never
  re-escape. An empty `highlighters` array falls back to the escaping

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [petersolopov/yace](https://github.com/petersolopov/yace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
