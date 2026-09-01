---
trigger: always_on
description: Project knowledge for working on the Blocks MailCraft editor. Host-facing API docs live in [README.md](README.md).
---

# AGENTS.md

Project knowledge for working on the Blocks MailCraft editor. Host-facing API docs live in [README.md](README.md).

## Workflow — always

- After any change under `src/` **or to README.md/DOCS.md**: `node build.js`. The demo page (`examples/vanilla.html`) loads `dist/mailcraft-editor.bundle.js`, never `src/` — a change isn't visible until the bundle is rebuilt. The same run regenerates the "Every message key" appendix in DOCS.md (from `src/core/i18n/en.js`, between the `message-keys` markers — never edit the table by hand) and the byte-exact `DOCS.md.txt`/`README.md.txt` mirrors the docs site serves raw (the "Copy for AI" button and `llms.txt`); `test/system.test.mjs` fails when any of them drift. Double-clicking the demo works: the example emails are inlined in it as `<script type="text/x-template">` blocks, so nothing is fetched. The same emails also live as standalone files in `examples/templates/` (byte-identical copies, for hosts to lift); those are not what the demo reads.
- `npm test` runs 13 suites (see the `test` script in package.json). `storage`, `export`, `templates`, `system` and `toolbar` are DOM-free and dependency-free; the rest (`editor-dom`, `css-cascade`, `render-dom`, `import-html`, `interaction`, `focus-preserve`, `shortcuts`, `import-structure`) drive the element and the importer on **jsdom** (a devDependency, alongside esbuild and c8). `npm run coverage` wraps the same suites in c8.
- The importer is covered by `test/import-html.test.mjs`, `test/import-structure.test.mjs` and `test/css-cascade.test.mjs` (jsdom). jsdom is not a real layout engine, so anything that depends on computed layout is still verified by hand through the Code modal — or by driving a scratch page through headless Chrome (`chrome.exe --headless=new --dump-dom`).
- Documents persist in `localStorage` **per browser tab** (`mailcraft.v3.tab.<id>`, seeded from the shared `mailcraft.v3`). An already-imported document keeps its old values — rebuilding does not retroactively fix it; re-import through the Code modal to see importer changes.
- Multiple sessions/tools may edit this repo concurrently — expect files to change on disk mid-session and re-read before editing. Source files carry **mixed CRLF/LF line endings**, so exact-match edits can fail on the newline alone.

## Build system constraints

`build.js` is a deliberate zero-dependency bundler with a regex transform. It only understands:

- `import { a, b as c } from '...'` (single-line)
- `export { a, b } from '...'`, `export const|function|class|async function NAME`

No `export default`, no bare `export { ... }`, no multi-line imports, no dynamic `import()`. A new file using anything else fails the build with "unhandled export syntax".

The generated bundle is then minified through esbuild (a devDependency) with an external sourcemap. `require()` is unavailable in `build.js` (the package is `type: module`) — it goes through `createRequire`. If esbuild is missing the build falls back to the readable bundle and deletes any stale `.map`, so a clone with no `node_modules` still produces a working `dist/`; that fallback is worth exercising after touching the build (`mv node_modules/esbuild node_modules/_off && node build.js`). Minifying buys ~30% raw and ~30% gzipped (245 KB → 172 KB) — the payload is mostly string data (the 31 i18n tables are ~371 KB of source on their own), which gzip compresses and a minifier cannot.

## Architecture

- `src/core/` — state and logic, **no DOM rendering** (EditorCore in `editor-core.js`: setState + rAF-batched emit; full re-render, no diffing). `src/render/` — all DOM building. The canvas is torn down and rebuilt per state change; focus/caret survive via `data-focus-key` + `render/focus-preserve.js`. Any new focusable field needs a stable focus key.
- Shadow DOM everywhere: `window.getSelection()` lies (use `core.getSelection()`), and window-level event listeners see retargeted `e.target` (the host element) — use `e.composedPath()[0]`.
- Inspector fields are descriptors (`core/binder.js`) rendered by `render/fields.js`. Toggle state flows through `on` → `aria-checked`. Number-like inputs are `type="text"` + `inputmode` on purpose (number inputs break caret restoration).
- Rows → columns → blocks. Row padding: linked `py`/`px` pair with optional per-side overrides `pt/pr/pb/pl` + `padSplit`; border sides `bTop/bRight/bBottom/bLeft`; `shadow` is a raw CSS string. Columns optionally carry `bg`/`radius`/`padY`/`padX`. `mkRow` defaults + `migrateDoc` backfill keep old saved docs valid — never make a renderer require a prop without a fallback.
- Export (`core/export.js`) reads back the rendered DOM (`grab`); renderer changes affect exported HTML directly. Import and export are meant to stay round-trip-compatible; `test/import-html.test.mjs` pins one round trip ("exported output re-imports"), but check anything beyond that shape by hand.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SELISEdigitalplatforms/blocks-mailcraft](https://github.com/SELISEdigitalplatforms/blocks-mailcraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
