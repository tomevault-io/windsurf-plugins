---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm install
npm run build          # full build -> dist/ ; also generates js/embedCss.js and stamps js/version.js
npm run updateCSS      # after editing css/*.scss: recompile SASS + regenerate js/embedCss.js
npm test               # mocha --ui tdd, runs test/*.js
npx eslint js          # lint (no npm script; config in eslint.config.js)
```

`js/embedCss.js` is generated from `css/igv.scss`, imported by `browser.js`, and not checked in — so it must exist before *anything* runs, including the Node tests. `npm install` covers this automatically (the `prepare` script runs `npm run build`); you only need to regenerate by hand after editing SCSS, or if you installed with `--ignore-scripts`. `js/version.js` is likewise rewritten from `package.json` on every build.

Tests use the **TDD interface** (`suite`/`test`, not `describe`/`it`) and **must be run from the repo root** — test data is referenced by repo-relative path (`test/data/...`).

```bash
npx mocha --ui tdd test/testBED.js              # single file
npx mocha --ui tdd test/testBED.js -g "BED query"  # single test
```

Every test file starts with `import "./utils/mockObjects.js"` (side-effect only), which installs globals — `document`, `window`, `File`, `XMLHttpRequest`, `DOMParser`, `atob`/`btoa` — so browser code runs under Node. `XMLHttpRequestMock` routes relative paths to the filesystem (with range-header support) and absolute URLs to the network, so most readers can be tested with no server. Genome fixtures come from `test/utils/MockGenome.js`.

## No bundler (a project-wide rule)

Every igvteam JavaScript project, igv.js included, must run **directly in a browser as ES modules, with no bundler, transpiler, or import map**. That is why third-party imports are written as explicit relative paths into `node_modules`:

```js
import {StringUtils} from "../node_modules/igv-utils/src/index.js"        // js/locus.js
import DOMPurify from "../../../node_modules/dompurify/dist/purify.es.mjs" // js/ui/components/textbox.js
```

New code must follow the same rule:

- Never write a bare specifier (`import ... from "igv-utils"`) — the browser cannot resolve it.
- Path the `../` prefix to the file's depth below the repo root, and point at the package's ESM entry file (`src/index.js`, `dist/*.es.mjs`, …), never at a directory or a `package.json` `exports` alias.
- Only add a dependency that ships an ESM build usable straight from `node_modules`; CommonJS-only packages cannot be used this way.
- Rollup (`npm run build`) exists to produce `dist/` for consumers — it is not a prerequisite for running the code.

## Development loop

Develop against the source, not `dist/`: the HTML files under `dev/` import `../js/index.js` directly as an ES module. Serve the repo root over HTTP and open e.g. `dev/igvjs.html`. `npm run build:dev-dashboard` regenerates `dev/dev.html`, a searchable index of every page under `dev/`.

CI (`.github/workflows/ci_build.yml`) runs `npm install && npm test` on Node 24.

## Architecture

Public API is `js/index.js` (the default export object); `igv.createBrowser(div, config)` lives in `js/igv-create.js`.

### Shadow DOM and CSS

`Browser` attaches a **shadow root** to the container div and adopts a stylesheet built from the generated `embedCss.js` string. igv.js styles are therefore fully isolated from the host page — and the reason a CSS edit is invisible until `npm run updateCSS` is run. Anything that queries the DOM must go through the shadow root, not `document`.

### Column layout (the key to multi-locus view)

`browser.columnContainer` is a flex row of columns created in a fixed order: axis, sample-info, sample-name, one `igv-column` **per locus** (separated by `igv-column-shim` elements), scrollbar, track drag handles, gear menu (gear can be moved to the left via `gearColumnPosition`). Multi-locus view is implemented by adding columns, not by nesting panels — see `viewportColumnManager.js`, `browser.addMultiLocusPanel`/`removeMultiLocusPanel`.

Each locus column is backed by a `ReferenceFrame` (`js/referenceFrame.js`) in `browser.referenceFrameList` holding `chr/start/end/bpPerPixel`. Panel widths are recomputed in `browser.calculateViewportWidth`.

### Track / TrackView / TrackViewport

Three distinct layers, easy to confuse:

- **Track** (`js/trackBase.js` + subclasses) — data + drawing, no DOM ownership. Holds config, state serialization (`getState`), menu items.
- **TrackView** (`js/trackView.js`) — one per track; owns that track's row across *all* columns: its axis canvas, scrollbar, drag handle, gear popup, and one `TrackViewport` per reference frame.
- **TrackViewport** (`js/trackViewport.js`) — one per (track, locus) cell; owns the canvas, the feature cache, mouse/click/popup handling, and per-cell SVG/PNG export.

Render pipeline: `browser.updateViews()` → collates `autoscaleGroup` tracks so a shared data range is computed across tracks before painting → `trackView.updateViews()` → each viewport decides `needsReload()` / `needsRepaint()` → `viewport.loadFeatures()` → `track.draw()`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [igvteam/igv.js](https://github.com/igvteam/igv.js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
