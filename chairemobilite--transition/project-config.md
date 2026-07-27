---
trigger: always_on
description: General context for Transition app development
---


# Transition Project - AI Context Guide

**This document provides comprehensive context for AI assistants working on the Transition codebase.**

## Project Overview

**Transition** is a transit planning and simulation platform developed by the Chaire Mobilité research group at Polytechnique Montréal. It's a full-stack web application for modeling, simulating, and planning public transit and alternative transportation systems.

- **Website**: http://transition.city/
- **Purpose**: Help transport planners (public or private) analyze transit networks, simulate scenarios, and plan transportation systems
- **License**: MIT
- **Repository**: github.com/chairemobilite/transition

---

## Architecture Overview

Transition is a **monorepo** using Yarn workspaces, structured as a full-stack TypeScript/React application with the following architecture:

```
transition-fork/
├── packages/                    # Main application packages (monorepo workspaces)
│   ├── chaire-lib-backend/     # Shared backend library
│   ├── chaire-lib-common/      # Shared common utilities
│   ├── chaire-lib-frontend/    # Shared frontend library
│   ├── transition-backend/     # Main backend application
│   ├── transition-common/      # Transition-specific shared code
│   └── transition-frontend/    # Main frontend React application
├── services/                   # Additional services
│   └── json2capnp/             # Rust service for Cap'n Proto caching
├── pyTransition/               # Python client library for Transition API
├── configs/                    # Shared configuration files (ESLint, Prettier, TypeScript)
├── locales/                    # i18n translation files (en, fr)
├── docs/                       # Documentation
├── examples/                    # Example configuration files
└── tests/                      # Integration tests
```

---

## Technology Stack

### Backend
- **Runtime**: Node.js
- **Language**: TypeScript
- **Framework**: Express.js
- **Real-time**: Socket.IO
- **Database**: PostgreSQL with PostGIS extension
- **ORM/Migrations**: Knex.js
- **Authentication**: Passport.js (local, bearer token)
- **Routing Engines**:
  - **OSRM**: For road network routing (walking, cycling, driving, bus modes)
  - **trRouting**: For public transit routing (primary engine)
- **Cache**: Cap'n Proto format (via Rust service `json2capnp`)
- **Build Tool**: TypeScript compiler (tsc)

### Frontend
- **Framework**: React with React Router
- **State Management**: Redux with Redux Thunk
- **Styling**: SCSS/SASS
- **Maps**: MapLibre GL JS with deck.gl for animations (uses OpenStreetMap tiles by default). Note: deck.gl requires WebGL support in the browser; ensure GPU/driver support is available. Consider providing a fallback or disabling animations when WebGL is unavailable.
- **Build Tool**: Webpack
- **UI Libraries**: React components (tabs, datepicker, modals, etc.)
- **Internationalization**: i18next (react-i18next)

### Additional Services
- **json2capnp**: Rust service for converting JSON transit data to Cap'n Proto format for high-performance caching
- **Memcached**: Used for caching (mentioned in Dockerfile)

### Development Tools
- **Package Manager**: Yarn (classic)
- **Linting**: ESLint with TypeScript plugin
- **Formatting**: Prettier (integrated with ESLint)
- **Testing**: Jest (unit tests), Playwright (UI tests)
- **Code Style**: Based on Google TypeScript Style (GTS) with customizations

---

## Package Breakdown

### `chaire-lib-backend`
Shared backend library containing:
- Database models and migrations
- API routing utilities
- Authentication/passport strategies
- OSRM and trRouting process managers
- File upload handlers
- Task management
- Internationalization setup
- User management

**Key directories:**
- `src/models/db/` - Database models (users, data sources)
- `src/api/` - API route utilities
- `src/config/` - Server configuration (knexfile, i18next)
- `src/utils/processManagers/` - OSRM and trRouting managers
- `src/scripts/` - Setup and migration scripts

### `chaire-lib-common`
Shared utilities used by both frontend and backend:
- Common data types and interfaces
- Utility functions (lodash extensions, GeoJSON helpers)
- Routing utilities
- Status/Result type handling

### `chaire-lib-frontend`
Shared frontend library:
- React components (reusable UI elements)
- Redux store configuration
- Map components
- Form components
- i18n configuration
- Styles

### `transition-backend`
Main backend application for transit planning:
- Transit-specific models (agencies, lines, nodes, paths, services, scenarios)
- GTFS import/export
- Transit routing calculations
- Accessibility map calculations
- Batch calculation jobs
- Simulations and evolutionary algorithms
- Public REST API (v1.1)
- Socket.IO API routes

**Key entry point**: `src/server.ts` - Main server initialization

**Key services:**
- `src/services/gtfsImport/` - GTFS data import
- `src/services/gtfsExport/` - GTFS data export
- `src/services/transitRouting/` - Transit routing calculations
- `src/services/simulation/` - Transit simulations
- `src/services/evolutionaryAlgorithm/` - Genetic algorithm for transit optimization
- `src/services/accessibilityMap/` - Accessibility map calculations
- `src/api/` - API routes (socket and REST)

**Database models:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chairemobilite/transition](https://github.com/chairemobilite/transition) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
