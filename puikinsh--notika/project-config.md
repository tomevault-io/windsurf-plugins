---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Notika** is a modernized admin dashboard template. Originally a Bootstrap 3 template from Colorlib, now running Vite 8.2.1 + Bootstrap 5.3.8 + ES6 modules. No jQuery.

## Commands

```bash
npm run dev        # Dev server with HMR (port 3100)
npm run build      # Production build to dist/
npm run preview    # Preview production build (port 4173)
npm run lint       # ESLint (flat config in eslint.config.js)
npm run lint:html  # html-validate across all 41 pages
npm run format     # Prettier
```

## Architecture

**Vite root**: `./notika/green-horizotal` (not project root). The `vite.config.js` lives at the project root but sets `root: './notika/green-horizotal'`.

```text
notika/green-horizotal/
├── src/js/
│   ├── main.js           # NotikaApp class - imports ALL CSS, main entry point
│   ├── modules/          # Shared modules (charts.js, ui.js)
│   └── pages/            # 31 page-specific modules extending NotikaApp
├── src/css/modern.scss   # Modern SCSS styles
├── src/partials/         # Handlebars partials (header.hbs, navbar.hbs, footer.hbs, breadcrumb.hbs)
├── css/                  # Template CSS (navbar-stable, mobile-menu, responsive, widgets)
├── *.html                # 41 template pages
└── img/                  # Images (Vite publicDir)
```

### How pages work

**Pages with custom logic** set `<html data-page-module="pagename">` and load a single script:

```html
<script type="module" src="/src/js/pages/pagename.js"></script>
```

The page module imports and extends `NotikaApp`. The `data-page-module` attribute prevents `NotikaApp` from auto-initializing (checked at `main.js:1181`).

**Pages without custom logic** omit `data-page-module` and load `main.js` directly, which auto-initializes `NotikaApp`.

### Page module pattern

```javascript
import { NotikaApp } from '../main.js'

class MyPage extends NotikaApp {
  async init() {
    await super.init()
    // page-specific setup
  }
}

if (document.readyState === 'loading') {
  document.addEventListener('DOMContentLoaded', () => {
    window.myPage = new MyPage()
    window.myPage.init()
  })
} else {
  window.myPage = new MyPage()
  window.myPage.init()
}

export { MyPage }
```

### Shared modules

- `modules/charts.js` — `NotikaCharts` class wrapping Chart.js. Stores instances in a `Map` for access/refresh/destroy.
- `modules/ui.js` — `NotikaUI` class for Bootstrap component init, counter animations (IntersectionObserver), toast notifications.

### CSS loading

All CSS is imported through `main.js` via Vite — both library CSS (Bootstrap, AOS, Leaflet) and template CSS files. Pages should NOT have `<link>` tags for CSS that Vite already bundles. Roboto is self-hosted via `@fontsource/roboto` (imported in `main.js`), so the only `<link>` tag in `<head>` should be the favicon. Do not reintroduce a Google Fonts CDN link — it is a third-party request and a GDPR concern.

### Standardized HTML head

```html
<html lang="en" data-page-module="pagename">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Page Title | Notika - Modern Vite Template</title>
    <meta name="description" content="...">
    <link rel="shortcut icon" type="image/x-icon" href="/favicon.ico">
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;500;700&display=swap" rel="stylesheet">
    <!-- Font Awesome and all other CSS imported via Vite -->
</head>
```

## Key Entry Points

**Main application**: `notika/green-horizotal/src/js/main.js`

- Imports all CSS through Vite (Bootstrap, AOS, Leaflet, template CSS, modern SCSS)
- Registers Chart.js components globally, sets `window.bootstrap`
- Font Awesome 7.2: tree-shaken via `library.add()` + `dom.watch()`
- `NotikaApp` class handles: charts, UI init, AOS animations, mobile menu, scrollbar cleanup

**Build config**: `vite.config.js`

- Multi-page app: all 41 pages in `rollupOptions.input`
- Manual chunks: `vendor` (Bootstrap), `charts` (Chart.js). Vite 8 builds with Rolldown, so `manualChunks` must be a **function**, and minification uses Oxc rather than esbuild.
- Page entry points are globbed from `notika/green-horizotal/*.html`; adding a page needs no config change.
- Production source maps are off.
- Handlebars plugin computes page context (title, nav active flags, breadcrumb icons) from filename
- SCSS uses `api: 'modern-compiler'`; PostCSS runs autoprefixer + cssnano
- Build target: `es2022`, output to `../../dist` (project root `dist/`)

## Adding New Pages

1. Create `notika/green-horizotal/newpage.html` using the standardized head pattern above
2. Add entry to `vite.config.js` → `rollupOptions.input`
3. Create `src/js/pages/newpage.js` extending `NotikaApp`
4. Set `<html data-page-module="newpage">` and load the page module script

## Adding Font Awesome Icons

Icons are tree-shaken. To use a new icon:

1. Import it in `main.js`: `import { faNewIcon } from '@fortawesome/free-solid-svg-icons'`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [puikinsh/notika](https://github.com/puikinsh/notika) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
