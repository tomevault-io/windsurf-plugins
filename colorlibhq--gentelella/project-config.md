---
trigger: always_on
description: Cross-tool agent instructions for Gentelella v4. Read by Aider, Cline, Codex, Continue, and any tool following the [agents.md](https://agents.md) convention. Claude Code reads `CLAUDE.md`; Cursor reads `.cursor/rules/`; GitHub Copilot reads `.github/copilot-instructions.md`. Content is intentionally overlapping — each tool only sees its own file.
---

# AGENTS.md

Cross-tool agent instructions for Gentelella v4. Read by Aider, Cline, Codex, Continue, and any tool following the [agents.md](https://agents.md) convention. Claude Code reads `CLAUDE.md`; Cursor reads `.cursor/rules/`; GitHub Copilot reads `.github/copilot-instructions.md`. Content is intentionally overlapping — each tool only sees its own file.

## What this is

Gentelella v4 (`4.0.0`) — free admin dashboard template by Colorlib. 58 server-rendered HTML pages in [production/](production/), built with **Vite 8** (Rolldown). **Vanilla ES2022**, no Bootstrap, no jQuery, no SPA framework. SCSS only. Heavyweight runtime deps are limited to **ECharts 6**, **DataTables.net 2**, and **Leaflet 1.9** — all lazy-imported per page.

Live preview: <https://preview.colorlib.com/theme/gentelella/>.

## Setup

```bash
npm install
npm run dev               # Vite dev server on :9173 → opens /production/index.html
```

Build / preview / deploy:

```bash
npm run build             # → dist/
npm run preview           # serve built dist/ on :9174
npm run deploy:preview    # build + sync to R2 with cache headers
```

## Architecture

- **Single entry** [src/main-v4.js](src/main-v4.js). Imports `scss/v4/main.scss`, mounts the shell, runs `initCharts/initTables/initCommandPalette/initPageActions`, then lazy-imports page-specific modules guarded by DOM presence (`if (document.getElementById('inbox-root')) import(...)`).
- **Shell injection at build time.** [vite.config.js](vite.config.js)'s `shellInjectionPlugin` inlines sidebar/topbar/footer into every page whose body has `data-shell="admin"`. No FOUC. Runtime [src/v4/shell.js](src/v4/shell.js) `mountShell()` is a fallback for opening raw HTML.
- **Auto-discovered entries.** `discoverEntries()` in [vite.config.js](vite.config.js) walks `production/*.html` and registers each as a Rollup input. No hand-maintained input list.
- **Three lazy vendor chunks**: `vendor-echarts` (chart pages), `vendor-tables` (table pages), `vendor-maps` (map page). Everything else ships in the main chunk.
- **NAV is one constant.** `NAV` in [src/v4/shell-render.js](src/v4/shell-render.js), 7 groups. Pages match into NAV by `data-page` ↔ leaf `key`.
- **Theming via CSS custom properties.** Tokens in [src/scss/v4/_tokens.scss](src/scss/v4/_tokens.scss) under `:root` and `[data-theme="dark"]`. Pre-paint inline script (in the Vite plugin) sets `data-theme` on `<html>` from `localStorage` before body renders.
- **PWA.** Service worker registered only in `import.meta.env.PROD`. `site.webmanifest` + meta tags injected into every page by the Vite plugin. Subpath-safe: paths use `import.meta.env.BASE_URL`.

## Directory layout

```text
src/
  main-v4.js               # Entry — mounts shell, lazy-loads modules
  scss/v4/                 # 10 partials, main.scss is the @use'd entry
  v4/
    shell.js               # mountShell — runtime shell behavior
    shell-render.js        # Pure renderers + NAV + ICONS
    menus.js               # openMenu / openPanel
    modal.js               # showModal
    toast.js               # showToast
    charts.js              # ECharts wrapper + factories
    tables.js              # DataTables wrapper
    command-palette.js     # ⌘K
    page-actions.js
    inbox.js kanban.js calendar.js settings.js file-manager.js
    form-controls.js       # Date range, multi-select, rich text
    details.js markup.js data-adapter.js
    product-images.js product-mockups.js
production/                # 58 HTML entry pages (auto-discovered)
public/                    # Copied verbatim to dist/
types/gentelella.d.ts      # Type declarations for the public JS surface
scripts/
  new-page.mjs             # npm run new -- <slug>
  screenshots.mjs          # npm run screenshots
  smoke.mjs                # npm run smoke
  deploy-preview.sh        # npm run deploy:preview
examples/                  # Standalone integrations (Express/SQLite, etc.)
```

## Conventions

1. **Vanilla DOM only.** `querySelector`, `classList`, `addEventListener`. No jQuery, no SPA framework.
2. **Lazy import per-page modules** with a DOM-presence guard so the main bundle never ships unused code.
3. **Idempotent `init<Name>()` exports.** Safe to call when the root element is absent; safe to call twice.
4. **Event delegation on `document`** for common interactions (toggles, todo checkboxes, chart tabs) — see the bottom of [src/main-v4.js](src/main-v4.js). Components that own their state (inbox, kanban, command palette) register on their own root.
5. **`showModal()` / `showToast()`** ([v4/modal.js](src/v4/modal.js), [v4/toast.js](src/v4/toast.js)) for overlays; **`openMenu()` / `openPanel()`** ([v4/menus.js](src/v4/menus.js)) for dropdowns and slide-outs. Both handle outside-click / escape / focus return.
6. **CSS custom properties for colors.** Never hex literals in components. Charts read them via `getComputedStyle(document.documentElement).getPropertyValue('--…')` so dark-mode redraw is automatic.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ColorlibHQ/gentelella](https://github.com/ColorlibHQ/gentelella) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
