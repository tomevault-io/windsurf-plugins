---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

### Initial Setup
```bash
# Build Arkime (C capture binary + install dependencies)
./easybutton-build.sh

# Load test data and run all tests
make check
```

### Running Tests
```bash
# Capture tests (protocol parsers, PCAP processing)
cd tests && ./tests.pl

# Viewer API tests (requires Elasticsearch running)
cd tests && ./tests.pl --viewer

# Viewer API tests without wiping ES and ingesting PCAP (requires Elasticsearch running)
cd tests && ./tests.pl --viewerstart

# Lint all code
npm run lint
npm run lint-fix  # Auto-fix issues
```

### Development Modes

Each component has a dev mode that runs frontend (Vite with HMR) + backend (nodemon) concurrently:

```bash
# Viewer (port 8123, anonymous mode)
npm run viewer:test

# Viewer (port 8123, with admin user)
npm run viewer:dev

# Cont3xt
npm run cont3xt:dev

# Parliament
npm run parliament:dev

# WISE
npm run wise:dev
```

### Building Frontend Bundles
```bash
# Production builds for each component
npm run viewer:bundle
npm run cont3xt:bundle
npm run parliament:bundle
npm run wise:bundle

# Minified builds (add :min)
npm run viewer:bundle:min
```

## Architecture Overview

Arkime is a modular network analysis system with these components:

### Core Components

**capture/** - C application (threaded, low-level)
- Monitors network traffic, writes PCAP files
- Parses packets, sends metadata to Elasticsearch
- Plugin system: parsers/ (protocols), plugins/ (capture methods)

**viewer/** - Node.js web application
- Express.js backend + Vue 3 frontend
- Provides web UI and API for browsing sessions
- Fetches PCAP data from capture nodes
- Main files: viewer.js (Node.js/Express.js layer), db.js (Elasticsearch layer)

**Elasticsearch/OpenSearch**
- Stores session metadata (SPI - Session Profile Information)
- Indices: sessions2-*, sessions3-*, partial-sessions3-*

### Optional Components

**cont3xt/** - Contextual intelligence gathering
- 39+ integrations (VirusTotal, Shodan, etc.) in integrations/
- Main file: cont3xt.js

**parliament/** - Multi-cluster monitoring and management
- Dashboard for managing multiple Arkime deployments
- Main file: parliament.js

**wiseService/** - Threat intelligence enrichment
- Pluggable sources (source.*.js files)
- Caching strategies: Redis, Memcached, LMDB, memory
- Main file: wiseService.js

### Shared Code

**common/** - Shared across all components
- arkimeConfig.js - Multi-format config system (.ini, .json, .yaml)
- auth.js - Authentication (digest, basic, OIDC, header, form)
- user.js - User management
- vueapp/ - Shared Vue components (Search.vue, Users.vue, etc.)
- vueapp/locales/ - i18n translations (9 languages + x-pl pseudo-locale: en, es, fr, de, ja, ko, zh, et, pt-BR)

## Frontend Architecture

### Tech Stack
- **Vue 3** - Composition API ready
- **Vite 8.x** - Modern bundler with HMR (replaces Webpack); uses Rolldown + Oxc internally
- **Bootstrap Vue Next** - Bootstrap 5 components
- **Vue Router 4** - SPA routing
- **Vuex 4** - State management
- **Vue i18n 11** - Internationalization

### Structure Pattern (using viewer as example)
```
viewer/vueapp/
├── src/
│   ├── main.js           # Vue app initialization
│   ├── router.js         # Routes (sessions, stats, users, hunt, etc.)
│   ├── store.js          # Vuex global state
│   └── components/       # Page-specific components
│       ├── sessions/     # ~19 components (list, detail, export)
│       ├── search/       # Query builder UI
│       ├── stats/        # Statistics dashboards
│       ├── spiview/      # SPI data visualization
│       ├── hunt/         # Saved searches
│       └── [others]/
└── dist/                 # Built output (generated)
```

All services follow this pattern. Common components live in common/vueapp/.

### Development Workflow
- Frontend: Vite dev server with hot reload (port configured in parentapp/vueapp/vite.config.mjs)
- Backend: nodemon auto-restarts on changes
- Proxy: Vite proxies API requests to backend

## Backend Architecture

### API Module Pattern

Each service uses Express.js with modular API files:

**Viewer API modules** (viewer/api*.js):
- apiSessions.js - Session search/retrieval
- apiStats.js - Aggregations and statistics
- apiUsers.js - User management/auth
- apiHunts.js - Saved searches
- apiConnections.js - Connection-based queries
- apiCrons.js - Scheduled searches

Pattern:
```javascript
class SessionAPIs {
  static async getSessions(req, res) { ... }
  static async buildSessionQuery(req, callback) { ... }
}
// Routes registered in main app
app.get('/api/sessions', SessionAPIs.getSessions);
```

### Database Layer

**db.js** in each service:
- Manages Elasticsearch/OpenSearch connections
- LRU caching for frequent queries
- Abstracts query building
- Index naming conventions

Key methods: Db.initialize(), Db.search(), Db.indexName(), Db.healthCheck()

### Configuration System

Unified config via common/arkimeConfig.js:
- Multi-format: .ini, .json, .yaml
- Environment variable overrides (ARKIME__section__key)
- Hierarchical sections with fallback
- Per-application customization


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arkime/arkime](https://github.com/arkime/arkime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
