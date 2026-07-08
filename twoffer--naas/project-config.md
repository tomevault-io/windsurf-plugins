---
trigger: always_on
description: **Full component specs, data flow, DB schema, Redis usage, communication patterns, and design rationale**
---

# CLAUDE.md — NAAS Project Reference

## Important Subdocuments

### 🏗️ [docs/architecture/SYSTEM_ARCHITECTURE.md](docs/architecture/SYSTEM_ARCHITECTURE.md) - Detailed Architecture

**Full component specs, data flow, DB schema, Redis usage, communication patterns, and design rationale**

### 🤖 [docs/AI-AGENT-PRINCIPLES.md](docs/AI-AGENT-PRINCIPLES.md) - Behavioral Guidelines

**CRITICAL:** These principles apply to ALL Claude Code sessions and sub-agents

Contains operational principles for rigorous software engineering:
- Assumption surfacing before implementation (never silently guess requirements)
- Confusion management (stop and clarify inconsistencies)
- Simplicity enforcement (resist overcomplication, prefer boring solutions)
- Scope discipline (surgical precision, no unsolicited refactoring)
- Push back when warranted (not a yes-machine, challenge bad ideas respectfully)

**When to use:** EVERY session, EVERY task, EVERY agent. These behavioral guidelines work alongside project-specific technical constraints.

**Key principle:** "You are the hands; the human is the architect. Move fast, but never faster than the human can verify."

## What Is This?

**NAAS (Normalized Adaptive Access System)** — Enterprise IAM modernization platform providing unified, risk-based access control across OIDC, SAML, and LDAP identity systems.

**Tagline:** "Normalize once. Secure everywhere."

## Tech Stack

- **Backend:** Python 3.12+ / FastAPI 0.115+ / SQLAlchemy 2.0 (async) / Pydantic 2.10+
- **Frontend:** React 19 / TypeScript / Vite 6 / shadcn/ui / Tailwind / TanStack Query / React Flow / Recharts
- **Data:** PostgreSQL 17+ / Redis 7.4+ (Streams, Pub/Sub, caching)
- **Infrastructure:** Docker Compose / Keycloak 26+ (OIDC) / OpenLDAP 2.6+ / Prometheus 2.54+ / Grafana 11+
- **Logging:** Structlog (JSON structured with correlation IDs)
- **ML:** scikit-learn (Random Forest ensemble)

## Project Structure

```
naas/
├── docker-compose.yml
├── CLAUDE.md                       # THIS FILE
├── services/
│   ├── api-gateway/                # JWT auth, routing, WebSocket, rate limiting
│   ├── event-ingestion/            # Accept + validate login events, dual-write PG + Redis
│   ├── identity-normalization/     # OIDC/SAML/LDAP adapters → unified schema ★ KEY DIFFERENTIATOR
│   ├── signal-enrichment/          # IP reputation, geo, device, impossible travel
│   ├── risk-evaluator/             # Rule-based + ML scoring → allow/MFA/deny
│   ├── policy-management/          # YAML policy CRUD, versioning, shadow mode
│   ├── alert-service/              # High-risk event alerting (never on historical events)
│   └── persona-simulator/          # LLM-powered event generation (Claude/Ollama/mock fallback)
├── scripts/
│   └── train_bootstrap_model.py    # ML model bootstrap — generates random_forest.pkl
├── shared/
│   └── naas_shared/
│       ├── ml_features.py          # ML feature column ordering contract
│       └── simulation_tools.py     # Tool definitions for persona-simulator and MCP server
├── config/
│   └── normalization.yaml            # Normalization service config: per-attribute authority weights, attribute importance, cross-protocol enrichment source config
├── dashboard/                      # React SPA (5 tabs + floating simulator panel)
├── infrastructure/                 # Docker configs: postgres, redis, keycloak, openldap, monitoring
└── docs/
    ├── architecture/
    │   └── SYSTEM_ARCHITECTURE.md  # ★ FULL architectural reference — READ THIS FOR DETAILS
    ├── adr/                        # Architectural Decision Records
    └── guides/
```

## Event Pipeline (async, Redis Streams)

```
Ingestion → [login_events] → Normalization (+ LDAP enrichment for OIDC/SAML) → [normalized_events] → Enrichment → [enriched_events] → Risk Evaluator
                                                                                                                                         │
Alert Service ◄── [decisions Pub/Sub] ◄── Risk Evaluator                                                                                 │
Dashboard     ◄── [alerts Pub/Sub]    ◄── Alert Service                                                                                  │
Dashboard     ◄── [decisions Pub/Sub] ◄──────────────────────────────────────────────────────────────────────────────────────────────────┘
```

## Key Commands

```bash
docker compose up              # Start all services
docker compose up -d           # Start detached
docker compose up -d --build   # Start detached, rebuilding local images (see note)
docker compose logs -f <svc>   # Tail service logs
docker compose ps              # Check service health
```

> **`--build` (locally-built images only):** `openldap` (`infrastructure/openldap/`) and every
> application service (e.g. `event-ingestion`) build from local Dockerfiles — a plain `up` reuses
> the cached image, so pass `--build` after changing their source. `postgres`/`redis`/`keycloak`
> pull pre-built images; `--build` is a no-op for them.
>
> **Schema changes need a volume wipe.** `infrastructure/postgres/init.sql` is the only schema
> source (no runtime DDL/migrations) and runs *only against an empty `postgres-data` volume*.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [twoffer/naas](https://github.com/twoffer/naas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
