---
trigger: always_on
description: Lightweight stateless markdown editor with live styling. Single Node.js file serves a single HTML file. No build step, no framework, one dependency (`marked` for MD parsing).
---

# SDocs

Lightweight stateless markdown editor with live styling. Single Node.js file serves a single HTML file. No build step, no framework, one dependency (`marked` for MD parsing).

## Stack

The repo holds two programs:

- **Server** (root `package.json`, `"private": true`, `name: sdocs-server`) - `server.js` plus everything outside `cli/`.
- **CLI** (`cli/package.json`, `name: sdocs-dev`) - published to npm. Zero runtime dependencies.

The two never share a `package.json` again. See "Published npm tarball" below for why this matters.

- **Server**: `server.js` - pure Node `http` module, small
- **CLI**: lives entirely under `cli/`:
  - `cli/bin/sdocs-dev.js` - the `sdoc` command (UMD-shared modules under `cli/shared/`)
  - `cli/bin/sdocs-postinstall.js` - global-install hint, no-op otherwise
  - `cli/shared/sdocs-yaml.js`, `sdocs-styles.js`, `sdocs-slugify.js` - real files (symlinked back into `public/` for the browser)
- **Frontend**: split across `public/`:
  - `index.html` - markup only
  - `css/tokens.css` - CSS custom properties, dark theme, theme transitions
  - `css/layout.css` - reset, body, topbar, main layout, left panel, divider
  - `css/rendered.css` - `#rendered` markdown styles, collapsible sections, copy buttons
  - `css/panel.css` - right panel, controls, statusbar
  - `css/mobile.css` - mobile `@media` breakpoint
  - `css/write.css` - write-mode contentEditable surface and toolbar
  - `css/comments.css` - comment-mode card / popover / gutter styling
  - `sdocs-yaml.js` - symlink to `../cli/shared/sdocs-yaml.js` (YAML front matter parse/serialize, UMD shared with Node)
  - `sdocs-slugify.js` - symlink to `../cli/shared/sdocs-slugify.js` (slugify heading text to URL-safe IDs, UMD shared with Node)
  - `sdocs-styles.js` - symlink to `../cli/shared/sdocs-styles.js` (pure style data tables + logic, UMD shared with tests)
  - `sdocs-state.js` - shared `window.SDocs` mutable state namespace
  - `sdocs-theme.js` - Google Fonts, font loading, dark mode, theme toggle
  - `sdocs-controls.js` - CSS variable management, color cascade, control wiring
  - `sdocs-chrome.js` - topbar / overflow menu / mobile sheet wiring
  - `sdocs-export.js` - PDF/Word/MD export, save-default styles
  - `sdocs-write.js` - write mode editor (contentEditable, toolbar, key handling)
  - `sdocs-charts.js` - Chart.js integration for ```chart fenced blocks
  - `sdocs-math.js` - KaTeX integration for `$$...$$` blocks
  - `sdocs-mermaid.js` - Mermaid integration for ```mermaid fenced blocks (lazy CDN, post-sanitised SVG)
  - `sdocs-mermaid-focus.js` - per-diagram fullscreen pan/zoom modal (drag, wheel, fit/100%/reset, ESC)
  - `sdocs-comments.js` - pure comment data model (anchor resolution helpers, YAML round-trip, footnote serializer), UMD shared with tests
  - `sdocs-comments-ui.js` - browser-only comment UI: rendering, selection popover, composer, navigation
  - `sdocs-cells.js` - pure cells data model (CSV parse, type classify, column names, format directive, selection stats, sort, header detection), UMD shared with tests
  - `sdocs-cells-formula.js` - pure formula engine for `=` cells (tokenize/parse/eval, refs + ranges, SUM/AVG/MIN/MAX/COUNT/COUNTA/PRODUCT/ROUND/ABS/IF, whole-model recalc with cycle detection), UMD shared with tests
  - `sdocs-cells-xlsx.js` - pure .xlsx writer (stored ZIP + SpreadsheetML, live formulas, number formats), zero dependencies, UMD shared with tests
  - `sdocs-cells-ui.js` - ```cells renderer: CSS-grid sheet (inline + fullscreen via a `fullscreen` flag), formula display, number/format rendering, copy toolbar, xlsx download, JS scroller sizing
  - `sdocs-cells-select.js` - cell + range selection and keyboard navigation for a cells grid
  - `sdocs-cells-focus.js` - fullscreen "focus" overlay for a sheet (name box, formula bar, selection stats footer); hosts the editor
  - `sdocs-cells-edit.js` - client-only in-cell editing for the fullscreen view (type/dblclick to edit, nav keys, undo/redo, delete-clear, TSV/CSV paste); mutates the shared model, never the document
  - `sdocs-app.js` - render orchestration, hash encode/decode, Brotli compression, syncAll, mode switching, drag/drop, file info card, scroll hints, init
  - `sdocs-info.js` - info panel, feedback link, notification dot
- **Tests**: `node test/run.js` - red/green, no test framework, uses Node `assert` + `http`
  - `test/runner.js` - shared harness: `test()`, `testAsync()`, `get()`, `report()`
  - `test/test-yaml.js` - YAML front matter parse/serialize tests
  - `test/test-styles.js` - SDocStyles pure module tests
  - `test/test-cli.js` - CLI parseArgs/buildUrl + style merging tests
  - `test/test-slugify.js` - slugify + heading dedup tests
  - `test/test-base64.js` - browser base64 UTF-8 roundtrip tests
  - `test/test-files.js` - file existence + content assertions
  - `test/test-http.js` - HTTP server tests (async); includes the per-route asset-versioning assertions
  - `test/test-cache-bust.js` - two-server check that asset URLs change when public/ contents change
  - `test/test-comments.js` - comment data-model + YAML/footnote round-trip + sanitisation tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [espressoplease/smalldocs](https://github.com/espressoplease/smalldocs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
