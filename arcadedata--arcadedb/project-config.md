---
trigger: always_on
description: This file provides guidance to Claude Code when working with the ArcadeDB Studio frontend.
---

# Studio CLAUDE.md

This file provides guidance to Claude Code when working with the ArcadeDB Studio frontend.

## Overview

ArcadeDB Studio is the web-based administration interface for ArcadeDB. It is a **traditional multi-page SPA** (not React/Vue/Angular) built with jQuery 4.0 + Bootstrap 5.3. All JavaScript modules use global functions and variables attached to the window scope. There is no module bundler for application code - only vendor libraries are processed by Webpack.

## Technology Stack

| Category | Library | Version |
|----------|---------|---------|
| DOM/AJAX | jQuery | 4.0 |
| UI Framework | Bootstrap | 5.3.8 |
| Code Editor | CodeMirror | 5.x (modes: SQL, Cypher, JavaScript) |
| Graph Visualization | Cytoscape.js | 3.33+ (plugins: fcose, cxtmenu, graphml, node-html-label) |
| Data Tables | DataTables | 2.3+ (extensions: buttons, responsive, select) |
| Charts | ApexCharts | 5.4+ |
| Notifications | SweetAlert2 | 11.x |
| Icons | FontAwesome Free | 7.2 |
| Export | JSZip (Excel), pdfmake (PDF) |
| API Docs | Swagger UI | 5.x |

**Constraint**: Only jQuery and Bootstrap 5 are allowed as core frameworks. Third-party libraries must be Apache 2.0 compatible (see root CLAUDE.md for allowed licenses).

## Build System

### npm + Webpack (vendor bundling only)

Webpack is used **solely** to copy vendor libraries from `node_modules/` into `src/main/resources/static/dist/`. It does NOT bundle, transpile, or process the Studio application JS files. The application JS files are loaded directly via `<script>` tags in `index.html`.

```
npm install          # Install dependencies
npm run build        # Webpack production build (copies vendors to dist/)
npm run dev          # Webpack watch mode for development
npm run clean        # Remove dist/ and node_modules/
npm run security-audit  # Comprehensive security audit
```

### Maven Integration

The `pom.xml` uses `frontend-maven-plugin` to run npm during Maven build:
- `npm install` runs during dependency resolution phase
- `npm run build` runs during `generate-resources` phase
- Node.js v18.19.0 is auto-installed by the plugin
- Build output is packaged into the JAR at `static/`

**Full build from project root**: `mvn clean install` (includes Studio build)
**Studio-only build**: `cd studio && npm run build`

### Build Output

```
src/main/resources/static/dist/    # Webpack output (vendor JS/CSS/fonts)
  js/         # ~30+ vendor JS files (minified)
  css/        # ~10 vendor CSS files
  webfonts/   # FontAwesome fonts
```

This folder is committed to git and deployed inside the JAR.

## Directory Structure

```
studio/
  src/main/
    js/
      vendor-libs.js              # Webpack entry point (minimal, just for webpack)
    resources/static/
      index.html                  # Main SPA entry point (loads all tabs)
      query.html                  # Query editor tab (SQL, Cypher, Gremlin, etc.)
      database.html               # Database schema management tab
      server.html                 # Server monitoring & metrics tab
      cluster.html                # Cluster/HA management tab
      api.html                    # Swagger API documentation tab
      resources.html              # Help & links tab
      popup-login.html            # Login modal dialog
      js/                         # Application JavaScript modules
        studio-utils.js           # Global utilities (alerts, cookies, HTML escape, formatting)
        studio-database.js        # Database listing, schema, create/drop/backup (LARGEST ~1350 lines)
        studio-server.js          # Server info, metrics charts, settings (~900 lines)
        studio-cluster.js         # Cluster status & HA monitoring (~240 lines)
        studio-table.js           # DataTables initialization & configuration (~170 lines)
        studio-graph.js           # Graph rendering setup, import/export (~250 lines)
        studio-graph-widget.js    # Cytoscape instance, layout, interactions (~930 lines)
        studio-graph-functions.js # Graph utility functions (~130 lines)
      css/
        studio.css                # Custom Studio styles
      images/                     # Logos, favicon, spinner, social icons
      dist/                       # Generated vendor assets (webpack output)
  scripts/
    copy-swagger-ui.js            # Copies Swagger UI files to static/
    security-audit.sh             # Security audit script
  package.json
  webpack.config.js
  pom.xml
  .nvmrc                          # Node 18.19.0
  .npmrc                          # legacy-peer-deps=true (for jQuery 4.0 compat)
```

## Application Architecture

### HTML Template System

`index.html` is the main entry point. It uses a server-side include directive `${include:filename.html}` to compose tabs at serve time. Each tab is a separate HTML file that gets inlined into the main page.

**Tab structure**: Left vertical sidebar nav (`col-1`) + Right content pane (`col-11`)

Tabs: Query | Database | Server | Cluster | API | Info | Logout

### JavaScript Module Loading Order

All modules are loaded via `<script>` tags in `index.html` (no ES modules, no imports). **Order matters**:

1. Vendor libraries (jQuery, Bootstrap, SweetAlert2, CodeMirror, DataTables, Cytoscape, ApexCharts)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ArcadeData/arcadedb](https://github.com/ArcadeData/arcadedb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
