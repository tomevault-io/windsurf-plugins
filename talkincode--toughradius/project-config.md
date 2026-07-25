---
trigger: always_on
description: When working on this repository (including any AI agent session, however it is run), align with these baselines:
---

# ToughRADIUS AI Coding Agent Instructions

## 📌 Project Guardrails (read first)

When working on this repository (including any AI agent session, however it is run), align with these baselines:

- **`AGENT.md`** — project-level AI development guide and working rules.
- **`docs/feature-checklist.md`** — feature scope baseline. Anchor every task to a `TR-F` ID; do not pursue non-goals `TR-N001`–`TR-N005`.
- **`docs/roadmap.md`** — long-term roadmap / milestones (the source of agent tasks).
- **`.agents/skills/<name>/SKILL.md`** — reusable task SOPs (orchestrate roadmap, review PR gate, groom roadmap, add vendor VSA, EAP method, admin API, React resource, config schema, acceptance test, sync upstream radius, reference RFC, align checklist, write Go tests). Pick the matching skill before starting.
- **Quality gates**: open a PR (never push `main`), and pass `go build ./...`, `go test ./...`, `golangci-lint run` (v2.12.2); for protocol/E2E changes add a CI acceptance test under `test/integration/`; for frontend changes run `cd web && npm run build`. Auto-delegation PRs are gated by the `review-pr` skill and auto-merged only when `agent-approved` + CI-green.
- **Protocol changes** must cite the relevant spec under `docs/rfcs/`.

## 🔍 Mandatory Requirements Before Development

**Never modify code blindly. Always gather context from the existing implementation, related tests, and documentation first.**

### When to Perform Context Retrieval

1. **Before Feature Development** – Locate similar features to mirror naming, error handling, and data flow
2. **Before Bug Fixes** – Trace the full execution path (handlers, services, models, DB access) before patching
3. **Before Refactoring** – Map dependencies and side effects to avoid regressions
4. **When Learning Conventions** – Study logging patterns, configuration helpers, and TDD expectations

### Recommended Search Techniques

```text
# Feature exploration
semantic_search "vendor attribute parsing" in internal/radiusd/vendors
file_search "*/internal/adminapi/*routes*.go"

# Bug fixing
list_code_usages AuthenticateUser
grep_search "AuthError" --include internal/radiusd/**

# Refactoring
semantic_search "errgroup" in main.go
grep_search "app.GDB" --include internal/app/**
```

**Core Principle: Understand existing code → Follow project conventions → Maintain consistency**

---

## Project Overview

ToughRADIUS is an enterprise-grade RADIUS server developed in Go, supporting standard RADIUS protocols (RFC 2865/2866) and RadSec (RADIUS over TLS). The frontend uses React Admin framework for the management interface.

## Architecture Highlights

### Core Service Concurrency Model

`main.go` uses `errgroup` to start multiple independent services concurrently. If any service crashes, the entire application exits:

- **Web/Admin API** - Echo framework, port 1816 (`internal/webserver` + `internal/adminapi`)
- **RADIUS Auth** - Authentication service, UDP 1812 (`internal/radiusd`)
- **RADIUS Acct** - Accounting service, UDP 1813 (`internal/radiusd`)
- **RadSec** - TLS-encrypted RADIUS over TCP, port 2083 (`internal/radiusd`)

### Project Structure Pattern

Follows golang-standards/project-layout:

- `internal/` - Private code, cannot be imported externally
  - `domain/` - **Unified data models** (all GORM models listed in `domain/tables.go`)
  - `adminapi/` - New management API routes (v9 refactor)
  - `radiusd/` - RADIUS protocol core implementation
  - `app/` - Global application instance (database, config, scheduled tasks)
- `pkg/` - Reusable public libraries (utilities, encryption, Excel, etc.)
- `web/` - React Admin frontend (TypeScript + Vite)

### Database Access Pattern

**Always** obtain GORM instance through `app.GDB()`, do not inject DB connection directly:

```go
// Correct
user := &domain.RadiusUser{}
app.GDB().Where("username = ?", name).First(user)

// Wrong - Don't do this
type Service struct { DB *gorm.DB }
```

Supports PostgreSQL (default) and SQLite (pure Go, no CGO). Database migration is automatically handled by `app.MigrateDB()`.

### Vendor Extension Handling

RADIUS protocol supports multi-vendor features, distinguished by `VendorCode` field:

- Huawei (2011) - `internal/radiusd/vendors/huawei/`
- Mikrotik (14988) - See `auth_accept_config.go`
- Cisco (9) / Ikuai (10055) / ZTE (3902) / H3C (25506)

When adding new vendor support, define constants in `radius.go`, then add switch cases in `auth_accept_config.go` and related processing functions.

## Key Development Workflows

### Build and Run

**Local Development** (SQLite supported, CGO disabled):

```bash
CGO_ENABLED=0 go run main.go -c toughradius.yml
```

**Production Build** (Supports PostgreSQL & SQLite, static compilation):

```bash
make build  # Output to ./release/toughradius
```

**Frontend Development**:

```bash
cd web
npm install
npm run dev      # Development server with hot reload
npm run build    # Production build, output to dist/
```

### Database Initialization

```bash
./toughradius -initdb -c toughradius.yml  # Drop and recreate all tables
```

Production environment uses `MigrateDB(false)` for automatic migration (configured in main.go).

### Testing Standards

- RADIUS protocol tests: `internal/radiusd/*_test.go`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [talkincode/toughradius](https://github.com/talkincode/toughradius) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
