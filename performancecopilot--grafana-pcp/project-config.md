---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the Performance Co-Pilot (PCP) Grafana Plugin - a Grafana app plugin that integrates PCP's performance monitoring capabilities into Grafana. The plugin provides:
- Three datasources that come bundled with premade dashboards (Valkey for historical metrics, Vector for real-time metrics, bpftrace for eBPF tracing)
- Three custom panels (Flame Graph, Troubleshooting Panel, Breadcrumbs)
- Premade dashboards for performance analysis using the USE (Utilization Saturation Errors) method

## Development Commands

### Setup
```bash
make deps                    # Install all dependencies (dashboards, frontend, backend)
make deps-frontend           # Install Node.js dependencies only
make deps-backend           # Install Go dependencies only
make deps-dashboards        # Install jsonnet dependencies only
```

### Building
```bash
make build                   # Build everything (dashboards, frontend, backend)
make build-frontend          # Build TypeScript/React frontend
make build-backend          # Build Go backend datasource
make build-dashboards       # Build Grafana dashboards from jsonnet

# Development builds
make dev-frontend           # Build frontend in development mode
make watch-frontend         # Auto-rebuild frontend on file changes
make dev-backend           # Build backend with race detector
make restart-backend       # Rebuild and restart backend (requires root)
```

### Testing
```bash
# Frontend tests
make test-frontend                  # Run frontend tests
make test-frontend-watch           # Run frontend tests in watch mode
make test-frontend-coverage        # Run frontend tests with coverage

# Backend tests
make test-backend                   # Run backend tests
make test-backend-coverage         # Run backend tests with coverage

# Run a single frontend test
yarn test <test-file-path>

# UI tests (Playwright)
make test-ui-start-pod             # Start PCP and Valkey in a pod
make test-ui                       # Run Playwright UI tests
yarn e2e                           # Run Playwright tests
yarn e2e:open                      # Open Playwright UI
```

### Other Commands
```bash
make zip                    # Create plugin zip file for distribution
make sign                   # Sign the plugin
make plugincheck           # Run Grafana plugincheck validation
make jsonnetfmt            # Format all jsonnet files
make clean                 # Remove all build artifacts
```

## Architecture

### Multi-Datasource Architecture
The plugin is structured as a Grafana **app plugin**. The app plugin is an umbrella plugin that bundles three distinct datasource plugins:
- **PCP Valkey** (formerly Redis): Backend datasource for querying historical metrics stored in Valkey/Redis using pmseries query language. This data source supports multihost monitoring, assuming the data for the hosts are stored in the same Valkey/Redis database.
- **PCP Vector**: Frontend datasource for real-time metrics from a single host using pmwebapi. Also has container support.
- **PCP bpftrace**: Frontend datasource for eBPF tracing scripts.

### Directory Structure
- `src/datasources/` - Frontend datasource implementations
  - `valkey/` - Frontend code for the PCP Valkey datasource
    - `dashboards/` - dashboards bundled with the PCP valkey datasource
  - `vector/` - frontend code for the PCP vector datasource
    - `dashboards/` - dashboards bundled with the PCP vector datasource
  - `bpftrace/` - frontend code for the PCP bpftrace datasource
    - `dashboards/` - dashboards bundled with the PCP bpftrace datasource
  - `lib/` - Shared datasource code (base classes, types)
- `pkg/datasources/valkey/` - Backend Go implementation for Valkey datasource
- `src/panels/` - Custom panel plugins (flamegraph, troubleshooting, breadcrumbs)
- `src/components/` - Shared React components (app page, search, Monaco editor)
- `src/common/services/` - API service clients
  - `pmapi/` - pmwebapi client for real-time metrics
  - `pmseries/` - pmseries API client for historical metrics
  - `pmsearch/` - Full-text metric search client
- `src/dashboards` - dashboards bundled with the pcp app plugin

### Key Architectural Patterns

**Backend Datasource (Valkey)**
- Uses Grafana Plugin SDK for Go (`grafana-plugin-sdk-go`)
- Implements `QueryDataHandler`, `CheckHealthHandler`, and `CallResourceHandler`
- Located in `pkg/datasources/valkey/`
- Communicates with pmproxy's `/series` endpoint
- Built as separate binaries for multiple platforms (see `build-backend` in Makefile)

**Frontend Datasources (Vector, bpftrace)**
- Inherit from `DataSourceBase<Q, O>` in `src/datasources/lib/pmapi/datasource_base.ts`
- Use shared PCP API services (`PmApiService`, `PmSeriesApiService`)
- Vector uses a sophisticated Poller (`src/datasources/lib/pmapi/poller/`) for real-time metric polling with:
  - Automatic derived metric registration
  - Instance and metric lifecycle management

**Service Layer**
- `PmApiService`: HTTP client for pmwebapi endpoints (context, indom, fetch, store, metrics)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [performancecopilot/grafana-pcp](https://github.com/performancecopilot/grafana-pcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
