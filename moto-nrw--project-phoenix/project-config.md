---
trigger: always_on
description: **Project Phoenix** - GDPR-compliant RFID student attendance and room management system.
---

# CLAUDE.md

## Project Overview

**Project Phoenix** - GDPR-compliant RFID student attendance and room management system.

| Component | Technology |
|-----------|------------|
| Backend | Go 1.23+, Chi router, BUN ORM |
| Frontend | Next.js 16+, React 19+, Tailwind 4+ |
| Database | PostgreSQL 17+ (multi-schema, SSL) |
| Auth | JWT (15min access, 7 days refresh) |

## Ecosystem

Project Phoenix is part of a three-repo system. All repos live side-by-side (`../`):

| Repo | Role | Relationship |
|------|------|-------------|
| **PyrePortal** (`../PyrePortal/`) | Raspberry Pi kiosk app (Tauri + React) | Consumes `/api/iot/*` endpoints with device API key + staff PIN auth |
| **moto-balenaOS** (`../moto-balenaOS/`) | Balena OS deployment layer | Runs PyrePortal + Phoenix backend on Raspberry Pi hardware |

**If you change IoT endpoints, error messages, or auth headers**: PyrePortal will break silently. Error messages are hardcoded in `PyrePortal/src/services/api.ts` and mapped to German UI text. Coordinate changes across repos.

### Presence mode (cross-repo contract)

`GET /api/iot/config` returns a `presence_mode: "detailed" | "binary"` field
that PyrePortal must respect. In `binary` mode the kiosk must hide room
selection, hide Raumwechsel/WC buttons, and branch the scan-result modal
based on `checkout.schulhof_enabled` (2-button door kiosk vs 3-button with
yard state). Missing or unknown values default to `detailed` so old kiosk
builds continue to work. Backend checkin semantics adapt transparently —
only the kiosk UI needs to change per mode. See the companion PyrePortal
issue for the exact UI state machine.

## Multi-Tenancy

### Tenant Hierarchy

```
Platform Operator (moto)
 └── Organization (Träger)           → platform.organizations
      └── School (OGS) = tenant      → platform.schools (school.id = tenant_id)
```

**School ID is the tenant boundary.** All 58+ tenant-scoped tables have a `tenant_id` FK to `platform.schools`. Account-to-school mappings live in `auth.account_tenants` (with lifecycle: pending → active → inactive).

### Scoping Mechanisms

| Layer | How |
|-------|-----|
| **JWT** | Claims include `tenant_id`, `org_id`, `scope` ("" = tenant, "org" = organization, "platform" = operator) |
| **Context** | `tenant.WithTenantID(ctx, id)` / `tenant.FromContext(ctx)` propagate tenant through request lifecycle |
| **Database** | `TenantTxMiddleware` sets PostgreSQL `LOCAL ROLE` + RLS config per request; auto-rollback on 5xx |
| **Models** | `base.TenantModel` (embeds `TenantID int64`) + `TenantScoped` interface on all tenant-aware entities |
| **Repositories** | `base.GetDB(ctx, db)` picks up tenant transaction; `base.EnsureTenantID(ctx, entity)` auto-populates tenant_id |

### Frontend Routing

- **Subdomain mode**: `{slug}.localhost:3000` → proxy rewrites to `/[tenant]/*` internally
- **Operator isolation**: `operator.localhost:3000` → rewrites to `/operator/*`, separate session
- **Tenant resolution**: `[tenant]/layout.tsx` validates slug via `/auth/tenant/resolve?slug=...` (cached 5min)
- **Tenant switching**: `POST /auth/switch-tenant` returns new JWT scoped to target school

### Key Env Vars

| Var | Purpose |
|-----|---------|
| `TENANT_DOMAIN` | Base domain for subdomain extraction (e.g., `localhost`, `moto-app.de`) |
| `NEXT_PUBLIC_TENANT_DOMAIN` | Client-side tenant domain |
| `NEXT_PUBLIC_OPERATOR_HOSTNAME` | Operator subdomain (e.g., `operator.localhost:3000`) |

### Reserved Slugs

Both backend (`models/platform/organization.go`) and frontend (`lib/reserved-slugs.ts`) maintain matching lists of reserved slugs (www, api, operator, grafana, etc.) that cannot be used as tenant subdomains. **These must stay in sync.**

### Cross-Repo Impact

Changing tenant resolution, auth headers, or error messages affects PyrePortal's device auth flow. The IoT API (`/api/iot/*`) uses device API keys (not tenant JWTs), but devices are scoped to schools.

## Core Architecture

**Handler → Service → Repository → Database** (always, no exceptions)

- `api/{domain}/` — HTTP handlers (thin, no business logic)
- `services/{domain}/` — Business logic, orchestration, transactions
- `database/repositories/{domain}/` — Data access only (BUN ORM)
- `models/{domain}/` — Domain entities, shared across layers
- Factory pattern for DI: `repositories.NewFactory(db)` → `services.NewFactory(repoFactory, db)`

## Critical Patterns

### 0. Frontend: Reuse Existing Components and Design Standards (MANDATORY)

**ABSOLUTE RULE: Before creating ANY new UI element, color, or component, search the existing codebase first.** Do not reinvent what already exists.

**Brand Colors** — MOTO uses specific brand hex codes, NOT generic Tailwind color classes. Before using any color:

1. **Check `frontend/src/lib/location-helper.ts` → `LOCATION_COLORS`** — the single source of truth for all semantic brand colors (green, blue, red, orange, purple, gray, amber). Read the file and use the exact hex values defined there.
2. **Check `frontend/src/contexts/ToastContext.tsx`** — established color patterns for success/error/info states.
3. **Check `frontend/src/styles/globals.css`** — logo gradient and other global color definitions.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moto-nrw/project-phoenix](https://github.com/moto-nrw/project-phoenix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
