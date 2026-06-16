---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

**Mash Potato** is a Postman-like desktop API client built with [Wails v2](https://wails.io/) — a Go backend bridged to a React/TypeScript frontend, compiled into a native desktop application.

## Commands

### Development

```bash
# Start development mode (hot-reload for both Go and frontend)
wails dev

# Build production binary
wails build
```

### Frontend only (from `frontend/`)

```bash
npm install       # Install dependencies
npm run dev       # Start Vite dev server standalone
npm run build     # TypeScript check + Vite build
npm run test      # Run Vitest suite
```

### Go backend

```bash
go build ./...    # Build Go packages
go vet ./...      # Lint Go code
go test ./...     # Run Go tests
```

### Makefile shortcuts

```bash
make dev          # wails dev
make build        # wails build
make test         # Go + frontend tests
make lint         # go vet
make generate     # Regenerate Wails JS bindings
make clean        # Remove build artifacts
```

## Architecture

### Stack
- **Backend**: Go 1.21, Wails v2, SQLite (`modernc.org/sqlite`)
- **Frontend**: React 18, TypeScript 5, Zustand (state), Vite (build), Vitest (tests)

### Communication
The frontend calls Go methods via Wails' auto-generated JS bindings in `frontend/src/wailsjs/go/`. These bindings are generated from the exported methods on the `App` struct in `app.go`. Any new Go method bound to `App` needs to be regenerated with `wails generate module` (or happens automatically on `wails dev`).

### Backend layout

| File/Dir | Purpose |
|---|---|
| `main.go` | Entry point; initializes DB, creates Wails app |
| `app.go` | `App` struct with all Wails-exposed methods |
| `interpolator.go` | `{{variable}}` template interpolation via regex |
| `curl.go` | cURL export (`ExportRequestAsCurl`) and import (`ImportFromCurl`) |
| `openapi_import.go` | OpenAPI 3.x YAML/JSON parser; scaffolds collections from parsed specs |
| `openapi_export.go` | Exports collections as OpenAPI 3.1 YAML (`ExportCollectionAsOpenAPI`) |
| `runner.go` | Collection runner; executes requests in sequence, emits `RunResult` events |
| `db/db.go` | SQLite init, WAL mode, foreign keys, schema migrations |
| `db/collections.go` | Collection CRUD |
| `db/requests.go` | Request CRUD with JSON-encoded headers/params |
| `db/folders.go` | Folder CRUD (nested folders within collections) |
| `db/environments.go` | Environment and variable CRUD |
| `db/history.go` | Request history CRUD |
| `db/settings.go` | App settings persistence (key-value) |
| `httpclient/client.go` | Executes HTTP requests; handles headers, query params, body types, response timing |
| `httpclient/assertions.go` | Test assertion evaluation against `ResponseResult` |
| `scripter/scripter.go` | JS pre/post-request script execution via `goja` |
| `encryption/vars.go` | AES-256 encryption/decryption for secret environment variables |

The SQLite database is stored in the OS user config directory (resolved at runtime).

**Database schema:**
- `collections(id, name, created_at)`
- `folders(id, collection_id, parent_folder_id, name, created_at)`
- `requests(id, collection_id, folder_id, name, method, url, headers, params, body_type, body, auth_type, auth_config, timeout_seconds, tests, pre_script, post_script, created_at)` — `headers` and `params` are JSON arrays of `{key, value, enabled}` objects.
- `environments(id, name, is_global, created_at)` — built-in "Global" environment seeded at startup (`id = '__global__'`)
- `environment_variables(id, environment_id, key, value, is_secret)`
- `settings(key, value)`
- `request_history(id, method, url, headers, params, body_type, body, response_status, response_body, response_headers, response_duration_ms, response_size_bytes, executed_at)`

### Frontend layout

| Path | Purpose |
|---|---|
| `frontend/src/App.tsx` | Root layout: Sidebar + TabBar + RequestEditor + ResponseViewer |
| `frontend/src/components/` | UI components (see below) |
| `frontend/src/store/` | Zustand stores (see below) |
| `frontend/src/types/` | TypeScript interfaces for `Collection`, `Request`, `Environment` |
| `frontend/src/hooks/` | Custom hooks |
| `frontend/src/utils/` | Shared utilities |
| `frontend/src/wailsjs/` | Auto-generated Wails bindings — do not edit manually |

**Stores:**

| Store | Purpose |
|---|---|
| `collectionsStore` | Collections list and CRUD actions |
| `requestsStore` | Open/active request state |
| `foldersStore` | Folder tree state |
| `tabsStore` | Open request tabs |
| `environmentsStore` | Environments and variables |
| `responseStore` | Last HTTP response + assertion results |
| `runnerStore` | Collection runner state and results |
| `historyStore` | Request history list |
| `settingsStore` | App settings (theme, timeout, etc.) |

**Hooks:**

| Hook | Purpose |
|---|---|
| `useVarAutocomplete` | `{{variable}}` autocomplete suggestions |
| `useCodeGen` | Code generation (curl, Python, etc.) from a request |
| `useDiff` | Diff computation between two responses |

**Utils:**

| Util | Purpose |
|---|---|
| `codeHighlighter` | Syntax highlighting for JSON and other response bodies |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NathanGdS/mash-potato](https://github.com/NathanGdS/mash-potato) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
