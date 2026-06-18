---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

```bash
make dev                # MySQL + build + setup + serve (one command)
make build              # Build UI + Go binary
make serve              # Start server on :8000
make restart            # Kill old server + rebuild all + start fresh
make setup              # Setup a site (SITE=airtime.local CONFIG=config/airtime/)
make test               # Run Go tests
make lint               # Run linters (Go + TypeScript)
make fmt                # Format code
make release TAG=v0.2.0 # Tag and push a release
make help               # Show all commands
```

### Manual Commands

```bash
# Backend (Go)
go build -o kora .                           # Build binary
go run . serve --port 8000                   # Dev run
go run . migrate --all                       # Apply all pending migrations
go run . config import --site X --path Y     # Re-import YAML config to DB

# Frontend (React SPA in ui/)
cd ui && bun install                         # Install deps
cd ui && bun run build                       # Build SPA → dist/
cd ui && bun run dev                         # Dev server (proxies /api → :8000)

# Docker
docker compose up -d mysql                   # MySQL 8.0 (root:kora123)
```

## Architecture

Kora is a **config-driven application engine**. Applications are YAML configs — the engine is generic and permanent. No code generation, no per-entity Go/React code.

### Startup Flow (`cli/serve.go`)

1. Load `common_site_config.yaml`
2. Discover sites in `sites/` (subdirs with `site_config.yaml`)
3. Per site: connect DB → bootstrap `_kora_*` tables → load config from DB → build Registry → run schema migration
4. Build `SiteRouter` (domain → site map)
5. Wire middleware: Recovery → RequestID → SecurityHeaders → CORS → SiteRouter → RateLimiter
6. Register auth routes (public), API routes (/api — SiteGuard), SPA (/workspace — NoRoute), console (/console — SystemGuard)
7. Start scheduler, listen, graceful shutdown on SIGTERM

### Middleware Chain

```
Request  → Recovery → RequestID → SecurityHeaders → CORS → SiteRouter → RateLimiter
         → API routes: SiteGuard (Auth + CSRF) → Permission check → Validation → ORM
         → /workspace: NoRoute handler serves SPA directly
         → /console: SystemGuard (system_credentials.yaml, separate from site auth)
         → /api/auth: public (no guard)
```

### Multi-Site Routing

Three methods coexist:
- **Host-based**: `Host` header → site (production, needs DNS)
- **Path-based**: `/s/:site/workspace` → site (dev, no config needed)
- **Default**: localhost/IP → first loaded site

The `SiteRouter` middleware sets `site_name`, `site_db`, `site_registry` in Gin context. **All auth is site-scoped** — login, session creation, session validation, and logout all read `site_db` from context. A session from one site doesn't work on another.

The `kora_site` cookie (set by path-based routing) is validated against the request Host header via `isHostAllowedForSite()` — only `localhost`, loopback IPs, or the site's configured domains are allowed. Unknown hosts get 403.

### API Envelope

All responses: `{"data": ..., "meta": {"doctype": "...", "total": N, "config_version": N}}`  
Errors: `{"error": "plain message"}` or `{"error": {"type": "UniqueConstraint", "message": "...", "field": "fieldname"}}`  
Multiple: `{"error": {"errors": [{"type": "...", "message": "...", "field": "..."}]}}`

**YAML validation errors** (from `POST /api/system/doctype/validate`):
```json
{"valid": false, "syntax": [{"line": 4, "column": 1, "key": "is_submittible", "context": "doctype", "detail": "did you mean \"is_submittable\"?"}]}
```
Unknown YAML keys are rejected with line numbers and Levenshtein-based suggestions. Fields inside `fields[]`, `constraints[]`, and `doc_constraints[]` are checked recursively.

### DocType & Field Config (`config/{app}/doctypes/*.yaml`)

Fields map to DB columns. Key field types: Data, Int, Float, Currency, Select, Link (autocomplete to target doctype), Table (child table — separate DB table with parent/parentfield/parenttype columns), Section Break, Column Break.

**New config-driven properties:**
- `computed: "quantity * unit_price"` — expression auto-calculated when dependencies change. Supports `+`, `-`, `*`, `/`, `SUM(table.field)`, `ROUND(expr, N)`
- `linked_field: "product.selling_price"` — auto-populates from linked document when Link field changes
- `unique: true` — DB UNIQUE index enforced at database level (MySQL error 1062 → field-level ValidationError)
- `renamed_from: "old_fieldname"` — non-breaking column rename via `ALTER TABLE RENAME COLUMN` during migration
- `constraints` — field constraints (min, max, regex, one_of, etc.) editable via visual form builder or YAML

### Frontend (`ui/`)

React 19 + TanStack Router/Query/Table/Form + shadcn/ui + Tailwind CSS v4 + Zustand. All views are **config-driven** — the SPA reads `/api/system/doctype/:name` and renders forms, lists, and workflow generically. No per-doctype components.

Key patterns:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [asenawritescode/kora](https://github.com/asenawritescode/kora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
