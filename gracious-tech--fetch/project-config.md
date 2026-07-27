---
trigger: always_on
description: Generates the service worker for the PWA app. Uses `StaleWhileRevalidate` strategy for
---


# fetch(bible) Platform

## Project Overview

fetch(bible) is a complete platform for digital Bible access, built by Gracious Tech. Unlike
typical API platforms, it uses a **client-side architecture** where the hosted portion is entirely
static files and the "API" is a client-side JavaScript module. This means no backend server is
needed — collections are hosted on a CDN and consumed directly by the client library.

**Key features:**
- A collector CLI that discovers, downloads, converts, and publishes Bible translations from
  multiple sources (eBible, DBL, etc.) into static file collections
- A client library (`@gracious.tech/fetch-client`) that consumes those collections in an API-like
  way, supporting browser and Node.js
- A full-featured Vue 3 Bible reading app with multi-translation view, search, cross-references,
  glosses, and study notes
- A web enhancer that detects Bible references on third-party pages and makes them interactive
- A search indexing library using flexsearch
- A Bible reference parsing/detection library (`@gracious.tech/bible-references`)
- Converters for Bible formats (USX to JSON, reverse versification)
- A VitePress documentation site at https://fetch.bible/

**Repository:** https://github.com/gracious-tech/fetch
**Website:** https://fetch.bible/
**License:** MIT-0 (MIT No Attribution)


## Architecture & Data Flow

### High-level flow

```
External sources (eBible, DBL, etc.)
        │
        ▼
   [collector]  ── discover → download → process → manifest → publish
        │
        ▼
   Static files on CDN (S3 + CloudFront)
        │
        ▼
   [client]  ── fetches manifest + translation data client-side
        │
        ▼
   [app / enhancer / third-party apps]  ── renders Bible content
```

### Component dependency graph (build order)

```
references  ─────────────────────────────────┐
    │                                        │
    ▼                                        ▼
  client  ──────────────────────────►  collector
    │                                    │
    ├──► enhancer                        │ (uses converters)
    │                                    │
    ▼                                    ▼
  search                          converters/usx-to-json
    │                             converters/reverse-usx
    ▼
   app
```

### Data formats

- **Source formats:** USFM, USX (XML), plain text — downloaded from external sources
- **Distribution formats:** JSON (HTML content per verse), JSON (plain text per verse), USFM, USX
- **Collection structure:** Static files with a manifest JSON at the root, translations organized
  by ID, chapters as individual files
- **Shared types:** Defined in `collector/src/parts/shared_types.ts` and symlinked into
  `client/src/assets/shared_types.ts`


## Tech Stack

| Technology | Purpose |
|---|---|
| **TypeScript** | Primary language for all packages (strict mode) |
| **Vue 3** | App and site UI framework (with Pug templates, scoped SASS) |
| **Vuetify 3** | Material Design 3 component library for the app |
| **Vite** | Dev server and bundler for app |
| **VitePress** | Static site generator for fetch.bible docs |
| **esbuild** | Fast bundling for libraries (client, search, references, converters, collector) |
| **vitest** | Test runner |
| **ESLint** | Linting (with Vue, TypeScript, Pug plugins) |
| **SASS** | Stylesheets for client CSS and enhancer |
| **yargs** | CLI argument parsing for collector |
| **flexsearch** | Full-text search indexing |
| **AWS S3 + CloudFront** | Hosting for collections, app, and site |
| **Workbox** | Service worker / PWA support in the app |
| **Bible Multi Converter** | External Java tool used by collector for format conversion |
| **@xmldom/xmldom + xpath** | XML parsing in collector for USX processing |


## File Structure & Entry Points

```
fetch_platform/
├── .bin/                    # Shell scripts for build, dev, publish, and tooling
│   ├── setup                # Full dev environment setup (npm ci + build all)
│   ├── setup_node           # Downloads project-scoped Node.js v25.8.1
│   ├── build_*              # Build scripts per workspace
│   ├── serve_*              # Dev servers (app, collection, site)
│   ├── audit_*              # Test, lint, type-check scripts
│   ├── test_collector       # Wipes and rebuilds a test collection end-to-end
│   ├── publish_*            # Publish scripts per workspace (S3 + npm)
│   ├── publish.ts           # Node.js S3/CloudFront publish utility
│   ├── collector             # Shortcut to run collector CLI
│   ├── switch_collections   # Swap main/alt test collections
│   └── upgrade              # Upgrade all workspace dependencies
├── app/                     # Vue 3 Bible reading app
│   ├── src/
│   │   ├── init.ts          # App bootstrap (Vue, Vuetify, collection loading)
│   │   ├── comp/            # Vue components (AppRoot, ContentInstance, NavPanel, etc.)
│   │   ├── services/        # State management, utilities
│   │   ├── css/             # Global styles
│   │   └── index.pug        # HTML template
│   └── vite.config.ts       # Vite config with Vue, Vuetify, SVG, Workbox plugins
├── client/                  # Core library for accessing collections
│   └── src/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gracious-tech/fetch](https://github.com/gracious-tech/fetch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
