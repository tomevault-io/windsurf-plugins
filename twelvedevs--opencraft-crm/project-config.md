---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Status

This is a **pre-sale planning repository** for **Ortho CRM** — an orthodontic-specific CRM platform. All 21 services (12 platform + 8 CRM + 1 frontend) are implemented. New services are built using the Ralph autonomous agent loop (see below).

## Key Documents

- `docs/00-prd-1.md` — Full PRD (v1.0, March 2026). Authoritative source for product decisions.
- `docs/01-platform-arch-design.md` — Platform architecture design (Draft, March 2026). Authoritative source for technical decisions.
- `docs/superpowers/specs/` — Component-level design specs (naming: `YYYY-MM-DD-{component}-design.md`). Authoritative for their named component once status is **Approved**.
  - `2026-03-24-automation-engine-design.md` — **Approved**
  - `2026-03-25-nurturing-engine-design.md` — **Approved**
  - `2026-03-25-messaging-service-design.md` — **Approved**
  - `2026-03-25-email-service-design.md` — **Approved**
  - `2026-03-25-notification-service-design.md` — **Approved**
  - `2026-03-25-template-service-design.md` — **Approved**
  - `2026-03-25-audience-engine-design.md` — **Approved**
  - `2026-03-25-ai-service-design.md` — **Approved**
  - `2026-03-25-analytics-service-design.md` — **Approved**
  - `2026-03-25-integration-hub-design.md` — **Approved**
  - `2026-03-25-media-service-design.md` — **Approved**
  - `2026-03-25-identity-service-design.md` — **Approved**
  - `2026-03-25-lead-service-design.md` — **Approved**
  - `2026-03-25-pipeline-engine-design.md` — **Approved**
  - `2026-03-25-conversation-service-design.md` — **Approved**
  - `2026-03-25-campaign-service-design.md` — **Approved**
  - `2026-03-25-reporting-service-design.md` — **Approved**
  - `2026-03-25-referral-service-design.md` — **Approved**
  - `2026-03-25-data-import-service-design.md` — **Approved**
  - `2026-03-25-crm-api-gateway-design.md` — **Approved**
  - `2026-03-25-crm-web-app-design.md` — **Approved**
- `docs/memories/` — Per-component key API/design decisions distilled from spec sessions (one file per service). Read the relevant file before working on or discussing a component.

## Architecture

Two-layer SOA with 21 independently deployable services in a Turborepo monorepo:

**Platform Layer (12 services)** — domain-agnostic, reusable across future products:
Messaging (Twilio), Email (SendGrid), Notification (SSE + Redis pub/sub), Template, Nurturing Engine (drip sequences), Automation Engine (event-driven workflows), Audience Engine (segment evaluation), AI (Claude API gateway), Analytics, Integration Hub (Google Ads + Meta), Identity (auth/RBAC), Media (S3/CloudFront)

**Product Layer — Ortho CRM (8 services)** — consume platform via REST + events:
Lead Service (core entity), Pipeline Engine (state machine), Conversation Service (SMS inbox), Campaign Service (email broadcasts), Referral Service, Reporting Service, Data Import Service (Ortho2 CSV), CRM API Gateway

**Frontend (1 app):** React 18 + TypeScript SPA at `apps/crm/web`. Embeds platform UI components (`@platform/template-ui`, `@platform/sequence-ui`, `@platform/audience-ui`, `@platform/automation-ui`) as React packages.

### Monorepo Structure

```
ortho/
├── apps/
│   ├── platform/        # 12 platform services (messaging, email, notification, etc.)
│   └── crm/
│       ├── lead/
│       ├── pipeline/
│       ├── conversation/
│       ├── campaign/
│       ├── referral/
│       ├── reporting/
│       ├── import/
│       ├── api-gateway/
│       └── web/         # React SPA
├── packages/
│   ├── @ortho/types         # shared TS interfaces for events + API contracts
│   ├── @ortho/event-bus     # typed EventBridge client (MockDriver available for tests)
│   ├── @ortho/auth-middleware
│   ├── @ortho/interpolator  # template variable interpolation
│   ├── @ortho/logger        # Pino, Datadog-compatible
│   ├── @platform/filter-engine  # shared pure-function filter evaluator (Automation + Audience engines)
│   └── @platform/audience-ui    # Audience segment builder React component
└── infra/               # IaC (AWS CDK or Terraform)
```

Each service follows: `src/{routes,services,repositories,events,queue}/` + `migrations/` + `test/{unit,integration}/` + `Dockerfile`

### Communication Patterns

**Async — AWS EventBridge** for state-change propagation (no direct coupling between publisher and consumer).

**Sync — REST** for queries and immediate commands (e.g. `POST /templates/render`, `POST /messages/send`, `POST /ai/complete`).

### Golden Rules (from arch doc)

1. Each service owns its DB schema — no cross-service table reads, all access through APIs or events.
2. Platform services never import product types — Automation Engine receives generic `{ entity_type, entity_id, event_type, payload }`.
3. Pipeline Engine only manages state — emits events; Automation Engine acts.
4. Platform UIs (`@platform/*`) call their own service's API directly from the browser (not proxied through CRM API Gateway). Auth uses the same Identity Service JWT.

## Local Development

Infrastructure (Postgres, Redis, Supabase Auth, MailHog) runs via Docker Compose. Before first run, generate crypto keys:

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [twelvedevs/opencraft-crm](https://github.com/twelvedevs/opencraft-crm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
