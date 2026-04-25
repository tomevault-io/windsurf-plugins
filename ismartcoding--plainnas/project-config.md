---
trigger: always_on
description: PlainNAS is a Go-based NAS (Network Attached Storage) system with a Vue 3 web frontend. The backend provides API, file system, media, and watcher services, while the frontend (in `web/`) offers a modern UI for user interaction.
---

# Copilot Instructions for PlainNAS

## Project Overview
PlainNAS is a Go-based NAS (Network Attached Storage) system with a Vue 3 web frontend. The backend provides API, file system, media, and watcher services, while the frontend (in `web/`) offers a modern UI for user interaction.

## Architecture & Key Components
- **Backend (Go):**
  - Entrypoint: `main.go`, with commands in `cmd/` (e.g., `run.go`, `install.go`).
  - Services: `internal/services/` (API, watcher), `internal/media/`, `internal/db/`, `internal/cache/`, `internal/config/`.
  - GraphQL: Defined in `internal/graph/schema.graphql`, resolvers in `internal/graph/schema.resolvers.go`, generated code in `internal/graph/generated/`.
  - Config: TOML files in `cmd/install/config.toml`.
  - Systemd integration: `cmd/install/plainnas.service`.
- **Frontend (Vue 3):**
  - Located in `web/`.
  - Main entry: `web/src/main.ts`, root component: `web/src/App.vue`.
  - Components: `web/src/components/`, assets: `web/src/assets/`.
  - Build tools: Vite (`vite.config.ts`), TypeScript (`tsconfig.json`).

## Developer Workflows
- **Install & Initialize:**
  - `sudo go run main.go install` (run once to set up packages, DB, config).
- **Run (Dev):**
  - `sudo go run main.go run`
- **GraphQL Codegen:**
  - `go env -w GOFLAGS=-mod=mod`
  - `go mod tidy`
  - `go generate ./internal/graph`
- **Production Build:**
  - `go build`
  - `sudo mv ./plainnas /usr/local/bin/`
  - `sudo systemctl start plainnas`
- **Frontend Build:**
  - From `web/`: `npm install && npm run build`

## Patterns & Conventions
- **Go Backend:**
  - Service boundaries: API, watcher, media, cache, DB, config are separated by directory.
  - GraphQL schema-driven development: Update `schema.graphql` and run codegen.
  - Config files (TOML) and systemd service for deployment.
- **Frontend:**
  - Vue 3 + TypeScript, Vite for builds.
  - Components are modular and located in `web/src/components/`.
  - Use `.env.local` for environment variables.

## Integration Points
- **API:** Go backend exposes GraphQL and REST endpoints (see `internal/services/api/`).
- **Watcher:** Monitors file changes (see `internal/services/watcher/`).
- **Media:** Handles media scanning/storage (see `internal/media/`).
- **Frontend-backend communication:** Via GraphQL and REST APIs.

## External Dependencies
- Go modules (see `go.mod`).
- Node.js packages for frontend (see `web/package.json`).
- Systemd for service management.

## Example: Adding a GraphQL Field
1. Update `internal/graph/schema.graphql`.
2. Run `go generate ./internal/graph`.
3. Implement resolver in `internal/graph/schema.resolvers.go`.

---

**For AI agents:**
- Always check for service boundaries before making changes.
- Use provided scripts and commands for setup/builds.
- Reference key files for patterns (see above).
- Ask for clarification if workflow or integration is unclear.

## Project Rules (Owner Requirements)
- Prioritize **simple, readable, minimal code** (less code is better).
- **Do not reduce features or logic**: behavior must remain correct.
- **Do not regress performance**: avoid full scans, avoid N+1 patterns, preserve index-backed fast paths.
- **Avoid compatibility/migration/backfill code** unless explicitly requested (deleting and rebuilding the DB is acceptable for development).
- **Avoid duplication**: extract shared helpers instead of repeating parsing/filtering/iteration logic.
- **Split oversized files**: keep files focused by responsibility (e.g., store vs indexes vs helpers).
- If logic/behavior changes, update relevant Markdown docs in `README.md` and/or `docs/`.
- Development workflow: it is acceptable to **delete the DB and rebuild**; avoid persistent index version machinery unless explicitly requested.

## Platform Support (Linux Only)
- PlainNAS is **Linux-only**. Design and implementation should assume Linux (systemd, `/proc`, `/sys`, `mount`, `lsblk`, etc.).
- Do **not** add Windows/macOS fallbacks, multi-platform abstractions, or non-Linux build stubs unless explicitly requested.
- It is acceptable for non-Linux builds to fail; correctness on Linux takes priority.

## Linux-Only File Naming
- Prefer **generic filenames** (e.g. `storage_mounts_list.go`) over `*_linux.go` suffixes for new files, since this repo is Linux-only.
- Only use OS-suffixed filenames when a build tag split is truly required (rare here) or when matching an existing established file pattern.

## VS Code / Copilot Rules
- If you change application behavior or logic, update the relevant Markdown docs in `docs/` (preferred) and/or `README.md` (keep docs in sync with code).
- Prefer minimal, readable, maintainable code; avoid unnecessary abstractions and duplication.
- Do not add compatibility/migration/backfill code unless explicitly requested; it is acceptable to require deleting/rebuilding the DB for development workflows.
- Prefer index-backed fast paths; avoid full scans and N+1 patterns.
- Prefer root-cause fixes over workaround logic (e.g., extra guards/dedup that mask bugs). If an invariant is broken, fix the source and let issues surface during development.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ismartcoding/plainnas](https://github.com/ismartcoding/plainnas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
