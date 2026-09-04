---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this
repository.

## Project Overview

Sufee is a static, front-end-only Bootstrap 5 admin dashboard template. No backend, no API
integrations. Multi-page HTML built with Vite, SCSS, and ES6 modules. 100% vanilla JavaScript (no
jQuery).

Version history lives in `CHANGELOG.md` — don't restate it here.

## Commands

```bash
npm run dev          # Vite dev server with HMR (default port 3001, override with VITE_PORT)
npm run build        # Production build to dist/
npm run preview      # Preview the production build
npm run lint         # ESLint with --fix
npm run lint:check   # ESLint without modifying files
npm run format       # Prettier --write
npm run quality:fix  # lint + format (run before committing)
npm run quality      # lint:check + format:check
```

`npm run typecheck` exists but is a stub — TypeScript is not configured.

## Architecture

### Two partials systems — don't confuse them

| System                       | Location                | Resolution                                          | Use for                                                      |
| ---------------------------- | ----------------------- | --------------------------------------------------- | ------------------------------------------------------------ |
| **Build-time** (Handlebars)  | `src/components/*.html` | Inlined at build via `vite-plugin-handlebars`       | Repeatable UI: cards, widgets — accept props                 |
| **Runtime** (fetch + inject) | `src/partials/*.html`   | Fetched from `./partials/` after `DOMContentLoaded` | Page chrome shared across every page (head, sidebar, header) |

Build-time partials are called inline: `{{> stat-card color="primary" value="42" ...}}`. Use
`{{prop}}` for HTML-escaped, `{{{prop}}}` for raw HTML (inline SVG, etc.). **Prettier mangles
multi-line partial calls** — `.prettierignore` excludes `src/components/` and `src/index.html` for
this reason.

Runtime partials are declared by attribute on a host element:

```html
<div data-partial="head-common"></div>
<div data-partial="sidebar" data-partial-replace="true"></div>
<div data-breadcrumb data-breadcrumb-title="Page" data-breadcrumb-path="Parent|Child|Page"></div>
```

### Entry point and init order (load-bearing)

`src/main.js` runs on `DOMContentLoaded`:

1. `new App().init()` — registers components, binds resize/keyboard listeners.
2. **`await partialsLoader.loadAllPartials()`** — fetches and injects every `[data-partial]` into
   the DOM. Awaited, not fire-and-forget — this is what makes step 3 deterministic.
3. `partialsLoader.initializeSidebarActiveState()`, `initializeHeader()`, `app.onPartialsReady()` —
   these all assume the sidebar/header DOM exists, which is now guaranteed by the await.
4. Dispatches a `partialsReady` event. User code can subscribe instead of polling.

There used to be `setTimeout(..., 100)` workarounds and per-partial event listeners to dance around
the race condition between partial injection and component setup. Those are gone — keep it that way.
If something needs the chrome to exist, await `loadAllPartials()` or subscribe to `partialsReady`.

### Component registration is selector-driven

`App.COMPONENT_REGISTRY` (in [src/scripts/app.js](src/scripts/app.js)) is an array of
`{ name, selector, load }` entries. On every page, the loader probes each `selector` against the DOM
and calls `load()` for matches. Components self-declare what DOM they own:

| Selector                                        | Component                                  |
| ----------------------------------------------- | ------------------------------------------ |
| `[id^="widgetChart"], #trafficChart, #worldMap` | `components/widgets.js` (WidgetManager)    |
| `[data-chart]`                                  | `components/charts.js` (ChartManager)      |
| `[data-table]`                                  | `components/datatable.js` (DataTable)      |
| `form[data-validate]`                           | `components/validation.js` (FormValidator) |

Adding a new component = append one entry to the registry. There's no central `data-page` switch any
more — `data-page` is purely for sidebar active-state highlighting.

### Build-time HTML entries are auto-discovered

`vite.config.js` runs `globSync('src/*.html')` to build the Rollup `input` map at config time. Drop
a new `.html` file in `src/`, run `npm run build`, it ships. No config edit needed. (`index.html` is
mapped to the `main` chunk for backwards compatibility.)

## Layout & styling

### Single source of truth for theme colors

Theme colors are SCSS variables in [src/styles/variables.scss](src/styles/variables.scss):
`$primary`, `$success`, `$danger`, etc. Bootstrap's `_root.scss` auto-generates the matching
`--bs-*` CSS custom properties.

**Reference colors as `var(--bs-primary)` everywhere** — never hardcode hex, never use a custom
`--sufee-*` or `--menu-*` namespace (those used to exist; they're gone). Layout/UI tokens that
Bootstrap doesn't generate (`--sidebar-*`, `--content-bg`, `--text-*`, `--header-height`) live in
the same file's `:root` block.

### Critical: do NOT import `bootstrap/dist/css/bootstrap.min.css`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [puikinsh/sufee-admin-dashboard](https://github.com/puikinsh/sufee-admin-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
