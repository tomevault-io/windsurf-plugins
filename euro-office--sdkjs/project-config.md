---
trigger: always_on
description: Guidance for AI agents working in **sdkjs** — the Euro-Office JavaScript SDK for the document
---

# sdkjs — Euro-Office JavaScript SDK

Guidance for AI agents working in **sdkjs** — the Euro-Office JavaScript SDK for the document
editors. This is the canonical instruction file; `CLAUDE.md` points here.

## What this repo is

sdkjs is the client-side engine for five document editors: **word** (text documents),
**cell** (spreadsheets), **slide** (presentations), **pdf** (viewer/annotator/editor), and
**visio** (diagrams). It is a hard fork of `ONLYOFFICE/sdkjs`, rebranded to Euro-Office, and
still periodically syncs from upstream (`git.onlyoffice.com`).

It is **not** an npm package. The build concatenates+minifies the sources into per-editor
bundles (`sdk-all.js` / `sdk-all-min.js`) that the sibling **web-apps** repo loads via
`<script>` tags. sdkjs lives in a multi-repo workspace alongside `web-apps` (the UI/toolbars),
`core` (the C++/WASM engine, fonts, x2t converter), `server`, and `DocumentServer`. Licensed
**AGPL v3** (GUI assets are CC-BY-SA 4.0).

## Repository layout

| Path        | Purpose |
| :---------- | :------ |
| `common/`   | Shared, editor-agnostic engine: base API, history/undo, co-editing, drawings, charts, fonts, formatting (DrawingML), serialization. Everything below depends on it. |
| `word/`     | Text-document editor + document model (`Editor/`), drawings, math, native overrides (`Local/`). |
| `cell/`     | Spreadsheet editor: `model/` (workbook + formula engine), `view/` (rendering), `graphics/`. |
| `slide/`    | Presentation editor + themes/textures. |
| `pdf/`      | PDF editor; `src/` is the implementation, `build/` a compiled wrapper, `test/` a harness. |
| `visio/`    | Diagram editor with its own VSDX serialization (`model/`). |
| `build/`    | Grunt build: `Gruntfile.js`, `package.json`, `license.header`. Run grunt from **here**. |
| `configs/`  | `<editor>.json` file-lists that drive the build (load order); `externs.json` for Closure. |
| `tests/`    | QUnit suites per editor + `code-style/check.py` (the lint gate). |
| `vendor/`   | Third-party libs (jQuery, XRegExp, etc.). Excluded from lint/build minification. |
| `tools/`    | One-off Python maintenance scripts (color schemes, SVG cursors). |

## Build & develop

For Docker dev environment setup (running the full server stack), see [/DocumentServer/AGENTS.md](../DocumentServer/AGENTS.md).

Requires **Node.js** and, for the full compile, **Java** (the build uses Google Closure
Compiler, pinned to `google-closure-compiler@20240317`). There is **no root `package.json`**;
all build deps live in `build/`.

```bash
# Full SDK build (release; ADVANCED minification). Run from build/.
cd build && npm install -g grunt-cli && npm ci && grunt
# Outputs: ../deploy/sdkjs/{word,cell,slide,visio}/sdk-all-min.js + sdk-all.js
```

```bash
# Debug/dev loop — NO recompile, NO Java needed. Run from build/.
grunt develop              # writes ../develop/sdkjs/<editor>/scripts.js listing the
                           # individual source files, so editors/tests load unminified sources
grunt develop --compiled   # same manifest, but pointing at the compiled bundles
```

Day-to-day inner loop: edit a source file → `grunt develop` → reload the editor/test page.
You only need the full `grunt` (Closure) build to produce release/min bundles.

Other flags: `--desktop=true` (desktop-only files), `--mobile=true`, `--map` (source maps),
`--level=WHITESPACE_ONLY` (faster, readable output), `--addon=sdkjs-forms` (merges an external
addon repo's `configs/`).

**`make` is NOT the SDK build.** The Makefile's default target also builds the sibling
`../web-apps` repo and requires it to be checked out next to sdkjs; it is the integration
build. Use `grunt` in `build/` for SDK-only work. (The Makefile's `SDKJS_FILES` is also stale —
it lists only `word/sdk-all.js` though grunt builds all editors.)

### Adding a source file

There are **no ES modules and no tree-shaking** — files are plain scripts concatenated in the
order listed in `configs/<editor>.json`. A new `.js` is invisible to the build until you add it
to the right array inside the top-level `sdk` object in the config (`sdk` → `min` = core engine
→ `sdk-all-min.js`; `sdk` → `common` = the rest → `sdk-all.js`; also `desktop`, `mobile`,
`exclude_mobile`). There is no flat key called `sdk.min` — grep for `"min":` inside the `sdk`
block. Order matters: dependencies must appear before dependents (e.g. `apiCommon.js`
before `api.js`).

## Tests

QUnit suites run headless via `node-qunit-puppeteer`, **from the repo root**:

```bash
# one-time setup (from repo root)
npm install grunt-cli node-qunit-puppeteer
npm install --prefix build
node node_modules/grunt-cli/bin/grunt --gruntfile build/Gruntfile.js develop

# run a single suite
node node_modules/node-qunit-puppeteer/cli.js tests/word/api/api.html 30000 "--no-sandbox"
```

CI (`.github/workflows/check-build.yml`, self-hosted runners) enumerates suites explicitly —
there is **no working aggregate runner** (`tests/runAll.js` exists but CI doesn't use it).

**CI branch scope:** `check-build.yml` triggers on push and pull_request for `fork`, `develop`,
`release/**`, and `hotfix/**`. It does **not** run on `main` — PRs targeting `main` are not
CI-guarded by this workflow.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Euro-Office/sdkjs](https://github.com/Euro-Office/sdkjs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
