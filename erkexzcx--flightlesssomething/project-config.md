---
trigger: always_on
description: FlightlessSomething is a full-stack web application for storing, managing, and visualizing gaming benchmark data. Users authenticate via Discord OAuth, upload MangoHud or Afterburner CSV benchmark files, and view interactive performance charts (FPS, frame time, CPU/GPU metrics). The app supports multi-user management, admin controls, audit logging, API token authentication, and an MCP (Model Context Protocol) server for AI assistant integration.
---

# Copilot Instructions for FlightlessSomething

## Project Overview

FlightlessSomething is a full-stack web application for storing, managing, and visualizing gaming benchmark data. Users authenticate via Discord OAuth, upload MangoHud or Afterburner CSV benchmark files, and view interactive performance charts (FPS, frame time, CPU/GPU metrics). The app supports multi-user management, admin controls, audit logging, API token authentication, and an MCP (Model Context Protocol) server for AI assistant integration.

**Live instances:**
- Production: `https://flightlesssomething.ambrosia.one/`
- Development: `https://flightlesssomething-dev.ambrosia.one/`

---

## Architecture

### Single Binary Deployment

The application compiles into a **single Go binary** with the Vue.js frontend embedded via Go's `//go:embed` directive. The `Makefile` builds the frontend first (`npm run build` → `web/dist/`), then compiles the Go server with the static assets baked in. At runtime, the binary serves both the API and the SPA from the same port (default `:5000`).

### Backend (Go)

- **HTTP framework:** [Gin](https://github.com/gin-gonic/gin) with session middleware
- **ORM:** [GORM](https://gorm.io/) with SQLite driver
- **Authentication:** Discord OAuth2 (via `golang.org/x/oauth2`) + session cookies + API Bearer tokens
- **Compression:** zstd (via `github.com/klauspost/compress`) for benchmark data storage
- **Encoding:** gob for binary serialization of benchmark data
- **Configuration:** CLI flags + environment variables (prefix `FS_`) via `peterbourgon/ff`
- **Entry point:** `cmd/server/main.go`
- **All application logic:** `internal/app/` package

### Frontend (Vue.js)

- **Framework:** Vue 3 with Composition API
- **State management:** Pinia (stores in `web/src/stores/`)
- **Routing:** Vue Router with eagerly imported views
- **Build tool:** Vite (dev server on port 3000, proxies API to port 5000)
- **Styling:** Bootstrap 5 + Font Awesome
- **Charts:** Highcharts for benchmark visualization
- **Security:** DOMPurify for HTML sanitization, Marked for Markdown rendering
- **Data loading:** `benchmarkRunLoader.js` parallel HTTP fetch for run data; `benchmarkDataProcessor.js` for lightweight format mapping; `statsCalculations.js` for percentile utilities (used by DebugCalc page)

### Data Storage

- **Database:** SQLite file at `{dataDir}/flightlesssomething.db`
- **Benchmark data files:** `{dataDir}/benchmarks/{id}.bin` (zstd-compressed gob, V2 streaming format)
- **Metadata files:** `{dataDir}/benchmarks/{id}.meta` (gob-encoded run count and labels)
- **Statistics files:** `{dataDir}/benchmarks/{id}.stats` (zstd-compressed gob, pre-calculated statistics)
- **Audit log:** `{parentOfDataDir}/logs/audit.json` (JSON lines, gzip-rotated at 10 MB, retains 10 rotated files)

---

## Directory Structure

```
flightlesssomething/
├── cmd/server/main.go              # Entry point: GC tuning, config load, server start
├── internal/app/                   # All backend application logic
│   ├── admin.go                    # Admin user management handlers (list/delete/ban/admin-toggle)
│   ├── api_tokens.go               # API token CRUD + RequireAuthOrToken middleware
│   ├── audit.go                    # File-based audit logging (JSON log with rotation)
│   ├── auth.go                     # Discord OAuth flow, admin login, session middleware
│   ├── benchmark_data.go           # CSV parsing, binary storage (V2), ZIP export, stats pre-calculation
│   ├── benchmark_stats.go          # Pre-calculated statistics types and computation
│   ├── benchmarks.go               # Benchmark CRUD handlers (create/read/update/delete/search)
│   ├── config.go                   # Configuration parsing (flags + env vars)
│   ├── database.go                 # GORM/SQLite initialization, admin user seeding
│   ├── debugcalc.go                # Debug calculation endpoint handler
│   ├── mcp.go                      # MCP server (JSON-RPC 2.0) with 10 tools + jq filtering
│   ├── migration.go                # Database schema versioning and migrations
│   ├── models.go                   # GORM models: User, Benchmark, APIToken
│   ├── ratelimiter.go              # In-memory sliding window rate limiter
│   ├── server.go                   # HTTP server setup, all route definitions
│   ├── storage_migration.go        # Benchmark data file format V1→V2 migration
│   ├── test_helpers.go             # Shared test utilities (setupTestDB, cleanupTestDB)
│   ├── web.go                      # Embedded SPA serving with fallback routing
│   └── *_test.go                   # Comprehensive test files (20 test files)
├── testdata/                       # Real benchmark CSV files for parsing tests
│   ├── afterburner/                # Afterburner HML format samples
│   └── mangohud/                   # MangoHud CSV format samples
├── web/                            # Vue.js frontend

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [erkexzcx/flightlesssomething](https://github.com/erkexzcx/flightlesssomething) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
