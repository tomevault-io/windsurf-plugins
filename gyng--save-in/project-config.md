---
trigger: always_on
description: A WebExtension that adds a context menu to save media/links/pages into
---

# save-in — agent/contributor guide

A WebExtension that adds a context menu to save media/links/pages into
chosen directories, with pattern-based routing and renaming. Ships to both
Firefox (AMO) and Chrome (Web Store).

This guide is the entry point for development. [docs/](docs/README.md) indexes
the deeper contracts it links to throughout.

## Architecture

**The code is ESM + TypeScript, shipped as readable, non-minified bundles.**
`config/rolldown.config.mjs` transpiles and scope-hoists each target into one file under
`dist/bundled/`. Production entries live in `src/entries/` for background,
options, offscreen, and reference-page targets; the content script is bundled
directly from `src/content/content.ts`. `background.e2e.ts` imports the
production background entry and adds the browser-test command only to e2e
builds.

`background/main.ts` is the background composition root. Source is grouped by
ownership under `automation`, `background`, `config`, `downloads`, `i18n`,
`menus`, `platform`, `routing`, and `shared`, plus the execution-context
directories `content`, `entries`, `offscreen`, `options`, and `vendor`. Only
`options/` is split into feature subdirectories, because only it grew to the
~120 files that made prefixes a poor index; `downloads/` and `shared/` are flat
by decision at ~33 each, where the biggest cluster is five files. Subdividing
them would rewrite import paths across a layer and every path-based rule in
`check-import-cycles.js` for a directory a reader can already scan — revisit if
a cluster grows, not before. The import graph is acyclic and
checked by `scripts/check-import-cycles.js`. Mutable cross-file state uses
explicit records or owner-controlled live bindings such as `options`,
`currentTab`, and `CURRENT_BROWSER`.

Bundle output is bare, scope-hoisted `esm` for background, options, and
offscreen classic contexts, and `iife` for isolated content and reference-page
scripts.

Options CSS is rooted at `src/options/style.css`, which declares the supported
cascade-layer order and imports ownership-oriented `style-*.css` files. Keep
production CSS as separate, readable source files; do not concatenate or
minify it into a generated production stylesheet. Keep feature rules with their
owner, preserve the declared layer order, and use the final utilities layer for
cross-feature state such as `[hidden]`. Use logical properties and
`text-align: start/end`, component container queries for owned workspace
responsiveness, and dynamic viewport-height units. All options boxes inherit
`border-box`; use semantic `--z-*` tokens instead of numeric `z-index`, contain
nested scroll surfaces, and preserve visible states in forced-colors mode.
The main options tabstrip must wrap at every viewport width and must never be a
horizontal or vertical scroll container; `scripts/check-css.js` enforces this.
Native nesting is allowed for short state/pseudo-element groups; do not build
deeply nested selector trees or create catch-all override stylesheets.
`@scope` is documented as a future migration in `docs/contributing/UI.md`; `scripts/check-css.js`
rejects it outright while Firefox below 146 remains supported (Chrome has it since
118, so the declared Firefox minimum is what gates it).

**Build, ship, and browser tests target the staged bundle** in
`dist/bundled-pkg`. `npm run typecheck` covers source and the TypeScript test
suite with `exactOptionalPropertyTypes` and `noUncheckedIndexedAccess`.

Execution contexts:

- **Background** (`src/background`, `src/downloads`, `src/config`, and shared
  feature directories): menus, download pipeline, messaging hub.
- **Content script** (`src/content/content.ts`): runs in every page;
  click-to-save and service-worker prewarming. Has no polyfill — uses
  callback-style `chrome.*` APIs, which work in both browsers.
- **Options page** (`src/options/*`): talks to the background exclusively
  via `runtime.sendMessage` (never `getBackgroundPage()`, which MV3 lacks).
- **Offscreen document** (`src/offscreen/offscreen.ts`, `src/offscreen.html`):
  Chrome-only. Lends the service worker a DOM so a fetched download becomes a
  blob object URL instead of a base64 data URL; it also hashes the same bytes
  and runs Prompt API calls.

The on-device rule assistant asks Gemini Nano for the facts of a request under a
response schema and assembles the rule text itself; it never asks the model to
write routing syntax, which it cannot do. Read
[docs/contributing/ON-DEVICE-PROMPT.md](docs/contributing/ON-DEVICE-PROMPT.md) before changing the prompts,
the response schemas, or the guardrails: what governs this model is measured
there, and prefer a schema change to a sentence — a sentence has never moved it.
Nothing reaches the rules editor until the deterministic guardrails, the
background `VALIDATE`, and the review all agree.

Automatic Page Sources saves reuse the normal `filenamePatterns` routing
language and editor. Eligibility is deliberately narrower than ordinary
routing: `routing/automatic-rule.ts` recognizes only rules with an explicit
`context` pattern for the synthetic `AUTO` context, plus at least one page
matcher and one source matcher. `automation/automatic-routing.ts` ignores all
other routing rules and selects the first eligible match. The content script

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gyng/save-in](https://github.com/gyng/save-in) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
