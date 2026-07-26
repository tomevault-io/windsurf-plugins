---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with the gbdraw web application.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with the gbdraw web application.

## Overview

`index.html` is the **SPA entry point** and loads ES modules from `gbdraw/web/js` (no build step). `gbdraw/web/js/app.js` mounts Vue and delegates to modules under `gbdraw/web/js/app/` (setup, Pyodide, run analysis, watchers, plus feature subfolders such as `legend/`, `legend-layout/`, and `feature-editor/`). The app runs gbdraw entirely in the browser using WebAssembly. No server is required - all genome data processing happens client-side via Pyodide (Python compiled to WebAssembly).

- **File size:** `index.html` contains HTML/CSS/templates; JavaScript lives under `gbdraw/web/js/`
- **Location:** `gbdraw/web/index.html`
- **Served by:** `gbdraw gui` command or hosted at https://gbdraw.app/

## Quick Reference

```bash
# Run locally
gbdraw gui                    # Opens browser at http://localhost:<free port>

# Prepare the generated local browser wheel (version must match pyproject.toml)
python tools/prepare_browser_wheel.py

# Refresh the cache-bust token when preparing a deployable web bundle
python tools/prepare_browser_wheel.py --refresh-cache-bust

# Build distributions after the browser wheel is prepared
python -m build

# Test in browser
# Open DevTools Console (F12) to see Pyodide output and errors
```

### Playwright Verification

- Do not skip browser verification solely because this repo has no local `node_modules/`, `package.json`, or `@playwright/test`. Check for Python/conda Playwright as well:
  - `command -v playwright && playwright --version`
  - `python -c "from playwright.sync_api import sync_playwright; print('python playwright ok')"`
- The JS specs in `tests/web/*.playwright.spec.js` need Node's `@playwright/test`; verify with `node -e "console.log(require.resolve('@playwright/test'))"` before running them.
- If `@playwright/test` is not installed, use Python Playwright for focused browser assertions against `gbdraw/web/index.html` or a local server.
- In Codex/agent sandboxes, Chromium may be installed but fail to launch with `sandbox_host_linux.cc ... Operation not permitted`. Request/run the browser check with the required sandbox escalation and report the real result.

### Key File References

| Section | File | Description |
|---------|------|-------------|
| CSP Header | gbdraw/web/index.html | Content Security Policy |
| CSS Styles | gbdraw/web/index.html | TailwindCSS custom classes |
| Vue App Template | gbdraw/web/index.html | HTML structure |
| App Entry | gbdraw/web/js/app.js | Mounts Vue and delegates to app setup |
| App Setup | gbdraw/web/js/app/app-setup.js | Composition root and module wiring |
| Pyodide Init | gbdraw/web/js/app/pyodide.js | Pyodide startup + wheel install |
| Run Analysis | gbdraw/web/js/app/run-analysis.js | Build args + execute gbdraw + extract features |
| Legend Management | gbdraw/web/js/app/legend.js | Legend actions (entry point; helpers under `gbdraw/web/js/app/legend/`) |
| Legend Layout | gbdraw/web/js/app/legend-layout.js | Legend/diagram positioning + canvas padding (entry point; helpers under `gbdraw/web/js/app/legend-layout/`) |
| Feature Editor | gbdraw/web/js/app/feature-editor.js | Feature color editor logic (entry point; helpers under `gbdraw/web/js/app/feature-editor/`) |
| Watchers | gbdraw/web/js/app/watchers.js | Vue watch hooks and lifecycle wiring |
| State Management | gbdraw/web/js/state.js | Reactive refs and computed |
| Components | gbdraw/web/js/components.js | HelpTip / FileUploader |
| Export Functions | gbdraw/web/js/services/export.js | PDF/PNG/SVG download |
| Config I/O | gbdraw/web/js/services/config.js | Save/load settings |
| PNG Helper | gbdraw/web/js/utils/png.js | DPI injection |

## Technology Stack

### Core Libraries (CDN)
| Library | Version | Purpose |
|---------|---------|---------|
| Vue.js 3 | 3.5.25 | Reactive UI framework |
| Pyodide | 0.29.0 | Python WebAssembly runtime |
| TailwindCSS | CDN | Utility-first CSS styling |
| jsPDF | 3.0.3 | PDF generation |
| svg2pdf.js | 2.6.0 | SVG to PDF conversion |
| DOMPurify | 3.2.7 | XSS protection for SVG output |
| Phosphor Icons | 2.1.2 | Icon library |

### Fonts
- Inter (Latin)
- Noto Sans JP (Japanese support)

## Architecture

```
index.html
├── <head>
│   ├── Content Security Policy (CSP)
│   ├── CDN script imports
│   └── TailwindCSS styles & custom CSS
├── <body>
│   ├── Vue app container (#app)
│   │   ├── Loading overlay (Pyodide init)
│   │   ├── Processing overlay
│   │   ├── Header (mode toggle, config save/load)
│   │   ├── Left Panel (4 columns)
│   │   │   ├── Input Genomes card
│   │   │   ├── Basic card
│   │   │   ├── Colors / Labels / Features (collapsible)
│   │   │   ├── Mode-specific Layout / Title&Legend / Axis & Scale / Pairwise Match cards
│   │   │   ├── Styles (collapsible)
│   │   │   ├── About & Citation (collapsible)
│   │   │   └── Generate button
│   │   └── Right Panel (8 columns)
│   │       ├── Error display
│   │       ├── Result preview (SVG container with zoom)
│   │       ├── Floating controls (zoom, padding, reset)
│   │       ├── Feature Color Picker Popup

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [satoshikawato/gbdraw](https://github.com/satoshikawato/gbdraw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
