---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture (2026 redesign)

This is the **Adminator 2026** template — a vanilla-JS admin dashboard with a token-driven CSS-variable design system. There is **one** entry bundle, **one** stylesheet root, **one** shell renderer.

- No jQuery. No Bootstrap. No legacy admin.js / Sidebar component.
- Theme (light/dark) is a `data-theme` attribute on `<html>`, set by an early-paint script in each page and toggled at runtime via `init.js`.
- Sidebar, topbar, and footer are rendered by `Shell.js` from a single `NAV` manifest. Pages provide three placeholder divs (`data-shell-sidebar`, `data-shell-topbar`, `data-shell-footer`) plus `<body data-active="..." data-crumbs="...">`.
- Heavy widgets: real **Chart.js** (`charts.js`), real **FullCalendar** (`calendar.js`), real **jsvectormap** (`maps.js`). All three read CSS variables at render and re-render on theme toggle via a `MutationObserver` on `data-theme`.
- **All three are lazy-loaded** via `import()`. Each `init*()` checks for its mount point and returns *before* the import when the page has no such element, so a page only downloads the library it actually uses. Every page's initial payload is `runtime.js` + `2026.js` + `style.css` (~130 KB raw / ~28 KB gzip) and nothing else.
- **No Babel.** webpack 5 parses ESM natively and the browserslist floor (Chrome/Firefox 90, Safari/iOS 15) needs no transpiling — an A/B build showed byte-equivalent output, so the toolchain was removed in 4.2.0. If you widen `browserslist` to older engines, reintroduce `babel-loader` in `webpack/rules/`.

## File layout

```text
src/
├── *.html                       # 18 pages — each ~500 lines, mostly content
├── assets/
│   ├── scripts/2026/            # The only JS
│   │   ├── index.js             # entry — imports SCSS, mounts shell, runs init
│   │   ├── Shell.js             # NAV manifest + sidebar/topbar/footer renderers
│   │   ├── init.js              # theme toggle, dropdowns, nav-groups, todos, accordions, tabs, mobile drawer
│   │   ├── palette.js           # ⌘K command palette — builds its list from NAV
│   │   ├── charts.js            # Chart.js SEEDS + tokens() — lazy-loads chart.js
│   │   ├── calendar.js          # FullCalendar seed events + toolbar binding — lazy-loaded
│   │   ├── maps.js              # jsvectormap world map — lazy-loaded
│   │   └── vendor-jsvectormap.js # re-export so lib + map data + CSS share one chunk
│   ├── styles/2026/             # The only SCSS
│   │   ├── index.scss           # entry — @use's every partial below
│   │   ├── _tokens.scss         # CSS variables, light + dark
│   │   ├── _base.scss           # reset, body, .eyebrow, .mono
│   │   ├── _animations.scss     # rise-in / bar-in / fade-in / draw / spin
│   │   ├── _shell.scss          # .shell, sidebar, topbar, footer chrome
│   │   ├── _dropdowns.scss      # .dd-* (notifications, messages, profile)
│   │   ├── _components.scss     # .hero, .btn, .card, .grid, .table, .tag
│   │   ├── _forms.scss          # inputs, select, textarea, check, radio, switch
│   │   ├── _ui.scss             # alerts, badges, progress, spinner, tabs, accordion, modal
│   │   ├── _auth.scss           # signin/signup split-screen shell
│   │   ├── _error.scss          # 404 / 500 cards
│   │   ├── _chat.scss           # 2-pane chat layout
│   │   ├── _data.scss           # data-table, pager
│   │   ├── _charts.scss         # chart-canvas-wrap, legend
│   │   ├── _dashboard.scss      # KPIs, sv-* (site visits), todo, weather
│   │   ├── _email.scss          # 3-pane email layout
│   │   ├── _calendar.scss       # mini-cal, rail, upcoming list
│   │   ├── _fullcalendar.scss   # FullCalendar token overrides
│   │   ├── _palette.scss        # ⌘K palette modal
│   │   └── _responsive.scss     # all media queries in one place
│   └── static/                  # 2.2 MB, currently referenced by NOTHING — see below
tests/                           # vitest + jsdom
webpack/                         # config, manifest, devServer, rules/, plugins/
docs/                            # Jekyll (just-the-docs) site — redirects to adminator.colorlib.com/docs/
scripts/                         # Playwright screenshot + publishing helpers (not part of the build)
```

The 2026 webpack entry produces two **initial** chunks — `runtime.js` and `2026.js` — which HtmlWebpackPlugin injects into every page, plus three **async** chunks fetched on demand: `vendor-chartjs.js`, `vendor-fullcalendar.js`, `vendor-jsvectormap.js` (+ its small CSS). In production, JS and CSS filenames get an 8-char contenthash; dev keeps plain names for readable source maps and HMR.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [puikinsh/Adminator-admin-dashboard](https://github.com/puikinsh/Adminator-admin-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
