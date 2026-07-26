---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# Agent Rules for evcc Project

This file provides guidance to AI coding agents when working with code in this repository.

## Project Overview

- evcc is an extensible EV Charge Controller and home energy management system written in Go with a Vue.js frontend
- The system manages electric vehicle charging, integrates with solar systems, and provides local energy management without cloud dependencies
- Architecture follows a plugin-based approach for device integrations

## Essential Commands

- `make` - build full application (UI + Go binary)
- `make build` - build Go binary only
- `make ui` - build UI assets only
- `make install` - install Go tools and dependencies
- `make install-ui` - install Node.js dependencies (`npm ci`)
- `make test` - run Go tests
- `make test-ui` - run frontend tests
- `make lint` - run Go linting (golangci-lint)
- `make lint-ui` - run frontend linting
- `npm run dev` - start Vue dev server (http://127.0.0.1:7071)
- `npm run playwright` - run integration tests
- `evcc --template-type [type] --template [file]` - test device templates
- `make docs` - generate template documentation

## Domain Knowledge

Deep documentation on specific subsystems is available in `docs/agents/`. Load what you need based on the task:

| File | When to load |
|------|-------------|
| [Core Domain](docs/agents/core-domain.md) | Control loop, loadpoint logic, PV surplus, charge modes, tariffs, interfaces |
| [Hardware Integrations](docs/agents/hardware-integrations.md) | Charger/meter/vehicle implementations, adding new devices |
| [Easee Architecture](docs/agents/easee-architecture.md) | Easee charger (REST+SignalR, async correlation, concurrency) |
| [OCPP Forwarder](docs/agents/ocpp-forwarder.md) | OCPP proxy/forwarder (sidecar relay to upstream OCPP server, read-only mode) |
| [Plugin System](docs/agents/plugin-system.md) | Plugin layer (HTTP, MQTT, Modbus, SunSpec, JS) |
| [Web UI & API](docs/agents/web-ui-api.md) | REST API, WebSocket, Vue frontend, authentication |
| [API Security](docs/agents/api-security.md) | Auth modes, JWT/API key/session, two-tier checks, credential storage |

### Loading guide by task type

- **Charger implementation** — hardware-integrations + core-domain
- **Easee charger work** — easee-architecture + core-domain
- **Meter implementation** — hardware-integrations + plugin-system
- **Vehicle implementation** — hardware-integrations
- **UI/frontend work** — web-ui-api
- **API endpoint work** — web-ui-api + core-domain
- **Auth / login / API key / permissions** — api-security + web-ui-api
- **Config/template work** — plugin-system
- **Control loop / charging logic** — core-domain
- **Bug in any area** — core-domain + relevant topic file(s)

## Architecture Guidelines

### Core Components

- **main.go** serves as entry point and embeds web assets and i18n files
- **cmd/** contains CLI commands, application setup, and various utility commands (configure, detect, migrate, etc.)
- **core/** contains core business logic with main files (loadpoint.go, site.go) and subdirectories:
  - **loadpoint/** - EV charging point management modules
  - **planner/** - Smart charging planning algorithms
  - **coordinator/** - Multi-loadpoint coordination logic
  - **session/** - Charging session management
  - **vehicle/** - Vehicle-specific core logic
  - **soc/** - State of charge handling
- **api/** contains API definitions and types
- **server/** handles HTTP server, WebSocket, MQTT, database operations, and various handlers
- **charger/**, **meter/**, **vehicle/** contain device integrations
- **tariff/** contains tariff integrations
- **plugin/** implements plugin system for device and tariff communication
- **assets/** contains Vue.js frontend application

### Frontend Structure

- **assets/js/** contains the main TypeScript/Vue.js application with:
  - **views/** - Vue page components (App.vue, Config.vue, Sessions.vue, etc.)
  - **components/** - Reusable Vue components
  - **composables/** - Vue utility functions
  - **types/** - TypeScript type definitions
  - **utils/** - Utility functions
  - **mixins/** - Vue mixins
- **assets/css/** contains application stylesheets
- **assets/public/** contains static assets and metadata
- **i18n/** contains internationalization files
- **tests/** contains Playwright integration tests and test configuration files
- **dist/** contains built frontend assets (generated)

## Writing Style

- No em dashes (—) in comments, commit messages, or docs. Use periods, commas, or colons
- Project name is `evcc`, always lowercase
- In user-facing strings, only mention `evcc` when needed to understand the context. Inside evcc's own UI the self-reference is usually redundant
- Acronyms uppercase in prose: OCPP, MQTT, HEMS, SoC
- Terminology: German "Phasensaldierung" (meter netting signed power across phases each instant) is "summative energy measurement" in English. Avoid "phase balancing" (means load balancing) and "net metering" (a billing scheme)
- Terminology: the top-level load management circuit is "root circuit" in English, "Hauptstromkreis" in German

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [evcc-io/evcc](https://github.com/evcc-io/evcc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
