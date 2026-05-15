---
trigger: always_on
description: Pad is a project management tool for developers and AI agents. Single Go binary with embedded SvelteKit web UI, SQLite storage, and multi-agent skill support (Claude Code, Cursor, Windsurf, Codex, Copilot, Amazon Q, Junie).
---

# Pad — Development Guide

## What This Is

Pad is a project management tool for developers and AI agents. Single Go binary with embedded SvelteKit web UI, SQLite storage, and multi-agent skill support (Claude Code, Cursor, Windsurf, Codex, Copilot, Amazon Q, Junie).

**Related repo:** The marketing website (getpad.dev) lives at `../pad-web` — a separate SvelteKit site deployed to Vercel.

## Architecture

- **Backend:** Go (cmd/pad/main.go) → REST API (internal/server/) → SQLite (internal/store/)
- **Frontend:** SvelteKit 2 + Svelte 5 (web/src/) → static build embedded in Go binary
- **Data model:** Workspaces → Collections (typed with JSON schemas) → Items (structured fields + rich content)
- **CLI:** Cobra commands in cmd/pad/main.go, HTTP client in internal/cli/
- **Agent skill:** Single natural-language `/pad` skill in skills/pad/SKILL.md

## Build & Install

```bash
make build      # Build web UI + Go binary (./pad)
make install    # Build, kill server, install to ~/.local/bin/pad, restart
make build-go   # Build Go only (skip web — faster when only backend changes)
make test       # Run Go tests
make web        # Build web UI only
make dev-web    # Run SvelteKit dev server (hot reload on :5173)
```

**After making changes, always run `make install`** to rebuild the binary, install it, and restart the server. The web UI at http://localhost:7777 will reflect the changes.

### Quick iteration loop

- **Backend only:** `make install` (skips web rebuild if no frontend changes — edit Makefile to use `build-go` instead of `build` in the install target)
- **Frontend only:** `make web && make install` or use `make dev-web` for hot reload during development
- **Full rebuild:** `make install`

## Key Directories

```
cmd/pad/main.go          — CLI entry point, all Cobra commands
internal/
  server/                — HTTP API handlers, SSE, middleware
  store/                 — SQLite CRUD, migrations, FTS
  models/                — Go types (Collection, Item, View, etc.)
  items/                 — Field validation against schemas
  collections/           — Default definitions, workspace templates
  cli/                   — HTTP client, formatting helpers
  events/                — EventBus for real-time SSE
  config/                — Workspace detection, .pad.toml
  diff/                  — Version diff storage
  webhooks/              — Webhook dispatcher with HMAC signing
  email/                 — Transactional email via Maileroo
  links/                 — Wiki-link parsing
web/src/
  routes/                — SvelteKit pages
  lib/api/client.ts      — TypeScript API client
  lib/types/index.ts     — TypeScript types
  lib/stores/            — Svelte 5 rune stores
  lib/components/        — Reusable UI components
skills/pad/SKILL.md      — Claude Code skill (embedded in binary)
```

## API

REST API at `/api/v1/`. Key endpoints:

- `GET/POST /workspaces/{ws}/collections` — collection CRUD
- `GET/POST /workspaces/{ws}/collections/{coll}/items` — item CRUD
- `GET/PATCH/DELETE /workspaces/{ws}/items/{slug}` — item by slug
- `GET /workspaces/{ws}/dashboard` — computed project overview (active items, plans, attention, blockers)
- `GET /workspaces/{ws}/activity` — workspace activity feed (enriched with item titles + change details)
- `GET/POST/DELETE /workspaces/{ws}/webhooks` — webhook management
- `GET /workspaces/{ws}/items/{slug}/children` — child items linked to a parent
- `GET /workspaces/{ws}/items/{slug}/progress` — child item completion progress
- `GET/POST /workspaces/{ws}/items/{slug}/links` — item relationships (blocks/blocked-by, parent/child)
- `GET /search?q=query&workspace=slug` — full-text search
- `GET /api/v1/events?workspace=slug` — SSE real-time events
- `GET /api/v1/collab/{itemID}?schema_version=N` — WebSocket upgrade for real-time collaborative editing (Yjs binary protocol; client must announce schema version)
- `GET /workspaces/{ws}/members` — list members + pending invitations
- `POST /workspaces/{ws}/members/invite` — invite user to workspace
- `GET /api/v1/auth/session` — auth status (`setup_required`, `setup_method`, `auth_method`, `authenticated`, `user`)
- `POST /api/v1/auth/bootstrap` — create the first admin account from localhost on a fresh instance
- `POST /api/v1/auth/register` — create account (admin-created or invitation-based after setup)
- `POST /api/v1/auth/login` — email/password login (returns session token)
- `POST /api/v1/auth/logout` — destroy session
- `GET/PATCH /api/v1/auth/me` — current user profile (GET) and update name/password (PATCH)
- `POST /api/v1/auth/forgot-password` — request password reset email
- `POST /api/v1/auth/reset-password` — reset password with token
- `GET/POST/DELETE /api/v1/auth/tokens` — user-scoped API tokens
- `GET/PATCH /api/v1/admin/settings` — platform settings (admin-only)
- `POST /api/v1/admin/test-email` — send test email (admin-only)
- `POST /api/v1/invitations/{code}/accept` — accept workspace invitation

## Authentication


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PerpetualSoftware/pad](https://github.com/PerpetualSoftware/pad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
