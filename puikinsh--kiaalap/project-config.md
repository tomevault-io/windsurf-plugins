---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Kiaalap is a Bootstrap 5 education-management dashboard: 62 static HTML pages composed from Handlebars partials at build/dev time by Vite. There is no application backend, router, or data layer — every page is hand-authored markup with page-local `<script>` blocks. It is fully jQuery-free (no jQuery in `package.json` or any source file).

## Commands

```bash
npm run dev          # Vite dev server on port 3000, auto-opens browser
npm run build        # Production build to dist/ (terser, drop_console)
npm run preview      # Serve the dist/ build
npm run lint:html    # html-validate on root *.html (config: .htmlvalidate.json)
npm run lint:css     # Stylelint on src/scss (config: .stylelintrc.json)
npm run lint:js      # ESLint on src/js/**/*.js
npm run format       # Prettier on src/**
npm run clean        # rm -rf dist
```

There is no test suite (`npm test` exits 1). All three linters and the build pass with zero errors and zero warnings — keep it that way.

## How a page is assembled

`vite-plugin-handlebars` renders every root `*.html` at request/build time. A page is a fragment that pulls in partials from `src/partials/`:

```html
{{> head}}      <!-- opens <!DOCTYPE html><html><head>…<body>, loads CSS -->
{{> sidebar}}
<div class="main-wrapper" id="mainWrapper">
    {{> header}}
    <main class="dashboard-content" id="main-content">…</main>
    {{> footer}}
</div>
<!-- page-local <script>/<link> tags go HERE, before the closing partial -->
{{> scripts}}   <!-- loads /src/js/main.js as a module, closes </body></html> -->
```

**`{{> scripts}}` must be the last thing in the file.** It emits `</body></html>`, so anything after it lands outside the document. Every page-local `<script>` or `<link>` belongs above it.

58 of 62 pages follow this. The four auth pages — [login.html](login.html), [register.html](register.html), [lock.html](lock.html), [password-recovery.html](password-recovery.html) — are standalone full documents with **no partials**; edits to `head.hbs`/`sidebar.hbs`/`header.hbs` do not reach them. [404.html](404.html) and [500.html](500.html) use `{{> head}}` and `{{> scripts}}` but deliberately render no sidebar.

Build inputs are globbed (`glob.sync('*.html')` minus `*template*` and `*-new*`), so a new root HTML file is included automatically.

## Loading model

1. **Bundled** — [src/partials/scripts.hbs](src/partials/scripts.hbs) loads `/src/js/main.js` as an ES module. `main.js` eagerly imports only what the shell and dashboard charts need on load — Bootstrap, Chart.js, dayjs — puts them on `window.*` for page-local inline scripts, and imports `layout.js`, `dashboard.js`, `charts.js`. Everything else is fetched on demand (see [On-demand libraries](#on-demand-libraries)).
2. **Page-local module scripts** — pages needing a library the bundle doesn't carry import it inside `<script type="module">`: Quill in [tinymc.html](tinymc.html), CropperJS in [images-cropper.html](images-cropper.html), Simple-DataTables in [data-table.html](data-table.html), Leaflet in the two map pages, Prism in [code-editor.html](code-editor.html). Vite bundles these into `dist/assets/`.

**No third-party script or stylesheet is loaded from a CDN.** Everything is served from the app's own origin, which is what lets the CSP in [DEPLOYMENT_GUIDE.md](DEPLOYMENT_GUIDE.md) stay strict. Adding a CDN `<script>` would break that — install the package and import it instead. The only remaining remote origins are demo image hosts and map tile servers.

Two rules follow from this:

- **Never add `<script src="node_modules/…">`.** A non-module script tag cannot be bundled, so it 404s in a `dist/` build. Use `<script type="module">` with a real `import`.
- **Chart.js is already global.** `window.Chart` is set by the bundle on every page. Page-local chart code just uses `new Chart(...)` inside a `DOMContentLoaded` handler — module scripts are deferred, so the global is set by the time that fires. Do not re-load Chart.js per page.

Inline `onclick="fn()"` attributes resolve against the global scope, which module scope is not part of. Pages that use them (`tinymc.html`, `images-cropper.html`) re-expose their handlers via `Object.assign(window, {...})`.

Leaflet and Prism still load from unpkg/cdnjs despite `leaflet` being an npm dependency.

## Handlebars context (vite.config.js)

`getPageContext(filename)` merges a base context (user, meta, currentYear) with a per-page entry from `pageConfigs`. Pages without an entry fall through to `defaultConfig`, which title-cases the filename.

Context keys that matter:

| Key | Consumed by | Purpose |
| --- | --- | --- |
| `page` | `sidebar.hbs` | `{{#if (eq page 'index')}}` marks the active link |
| nav group booleans | `sidebar.hbs` | `{{#if academic.students}}` expands the containing submenu |
| `breadcrumb` | `header.hbs` | array of `{title, url}` |
| `title`, `pageTitle`, `pageDescription`, `showPageHeader` | `head.hbs` / pages | |
| `additionalCSS` | `head.hbs` | extra `<link>` tags, paths relative to project root |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [puikinsh/kiaalap](https://github.com/puikinsh/kiaalap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
