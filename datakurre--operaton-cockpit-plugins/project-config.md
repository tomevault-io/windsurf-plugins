---
trigger: always_on
description: This repository bundles minimal history-oriented plugins for Operaton and Camunda 7 Cockpit/Tasklist. TypeScript + React sources in `src/` are bundled with Rollup into top-level `*.js` files that Cockpit loads via `config.js`.
---

# AGENTS Guide

This repository bundles minimal history-oriented plugins for Operaton and Camunda 7 Cockpit/Tasklist. TypeScript + React sources in `src/` are bundled with Rollup into top-level `*.js` files that Cockpit loads via `config.js`.

**Note:** Built JavaScript modules are committed to the repository for convenience. Users can deploy the plugins directly without running the build process.

**Plugin status legend** used throughout this document:

| Status | Meaning |
|--------|---------|
| **shipped** | Referenced from a `*config.js` and/or bundled by the [Dockerfile](Dockerfile). This is what users actually run. |
| **optional** | Built and committed, but not referenced by the default configs. Users opt in by adding it to their own `config.js`. |
| **abandoned** | Sources and bundle are kept for reference only. Not referenced by any config, not bundled by the Dockerfile, and not maintained. Do not invest in it. |

A source file existing under `src/` does **not** mean the plugin is shipped — always cross-check
[config.js](config.js), the other `*config.js` files, and the [Dockerfile](Dockerfile).

## Project map

### Configuration and deployment
- [README.md](README.md): Usage and deployment instructions
- [config.js](config.js): Cockpit plugin configuration (defines `customScripts` and `bpmnJs.additionalModules`)
- [cockpit-nologin-config.js](cockpit-nologin-config.js): Alternative Cockpit configuration that only loads `cockpit-nologin.js`
- [tasklist-config.js](tasklist-config.js): Tasklist configuration (`tasklist-audit-log.js`, `tasklist-custom-styles.js`)
- [admin-config.js](admin-config.js): Admin configuration (`admin-route-authorization.js`, `admin-custom-styles.js`)
- [welcome-config.js](welcome-config.js): Welcome configuration (`welcome-custom-styles.js`)
- [Dockerfile](Dockerfile): Standalone Operaton Docker image build (context-free); its `cp` list is the
  authoritative statement of which bundles ship

### Build pipeline
- [rollup.config.mjs](rollup.config.mjs): Compiles each plugin entrypoint in `src/` to a top-level `*.js` bundle
- [package.json](package.json): Dependencies and npm scripts
- [tsconfig.json](tsconfig.json): TypeScript configuration
- [Makefile](Makefile): Development shortcuts (formatting, etc.)

### Built outputs (committed for convenience)
- `admin-custom-styles.js` – Custom stylesheet plugin for Admin UI
- `admin-nologin.js` – Admin no-login plugin (hides signin form via CSS)
- `admin-route-authorization.js` – Admin authorization management route
- `cockpit-custom-styles.js` – Custom stylesheet plugin for UI customization
- `cockpit-nologin.js` – Cockpit no-login plugin (hides signin form via CSS)
- `dashboard-favourites.js` – Process definition favorites star button and dashboard table
- `dashboard-integrations.js` – External task ("integrations") dashboard with retry/unlock actions
- `decisions-dashboard.js` – **abandoned** DMN decision simulator (see [Abandoned plugins](#abandoned-plugins))
- `definition-historic-activities.js` – Process definition statistics overlay
- `definition-tab-modify.js` – Process definition modification template builder
- `instance-action-unlock.js` – External task unlock action
- `instance-auto-refresh.js` – Auto-refresh toggle for instance views
- `instance-historic-activities.js` – Process instance audit log and overlays
- `instance-route-history.js` – Full history route view
- `instance-tab-modify.js` – Process modification and message correlation
- `robot-module.js` – BPMN module utilities for diagram rendering
- `tasklist-audit-log.js` – Tasklist audit log tab
- `tasklist-custom-styles.js` – Custom stylesheet plugin for Tasklist UI
- `tasklist-nologin.js` – Tasklist no-login plugin (hides signin form via CSS)
- `welcome-custom-styles.js` – Custom stylesheet plugin for Welcome UI
- `welcome-nologin.js` – Welcome no-login plugin (hides signin form via CSS)
- `*.js.map` – Source maps (development builds only)

### Plugin entrypoints (`src/`)
- [src/admin-custom-styles.tsx](src/admin-custom-styles.tsx): Minimal plugin that applies custom stylesheets (SCSS) for Admin UI customization.
- [src/admin-nologin.tsx](src/admin-nologin.tsx): Admin no-login plugin that hides the signin form with CSS. For environments with external authentication (SSO, reverse proxy).
- [src/admin-route-authorization.tsx](src/admin-route-authorization.tsx): Admin authorization management route. Two-panel layout with resource type list and authorization CRUD operations.
- [src/cockpit-custom-styles.tsx](src/cockpit-custom-styles.tsx): Minimal plugin that only applies custom stylesheets (SCSS) for UI customization without any JavaScript functionality.
- [src/cockpit-nologin.tsx](src/cockpit-nologin.tsx): Cockpit no-login plugin that hides the signin form with CSS. For environments with external authentication (SSO, reverse proxy).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [datakurre/operaton-cockpit-plugins](https://github.com/datakurre/operaton-cockpit-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
