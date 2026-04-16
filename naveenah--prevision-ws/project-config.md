---
trigger: always_on
description: > The Brain: loaded first on every Copilot interaction. Keep focused on non-negotiable rules and project identity.
---

# AI Brand Automator — Copilot Instructions

> The Brain: loaded first on every Copilot interaction. Keep focused on non-negotiable rules and project identity.

## Project Identity

AI Brand Automator is a **multi-tenant SaaS platform** for AI-powered brand building. Users onboard a company, upload brand assets, and the platform generates brand strategies, manages social media, schedules content, and integrates Google Business Profiles — all powered by Gemini 2.0 Flash.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js 15, React 19, TypeScript (strict), Tailwind CSS v4 |
| Backend | Django 4.2, Django REST Framework, SimpleJWT |
| AI | Google Gemini 2.0 Flash (`GeminiAIService` singleton), Anthropic Claude (brand equity) |
| Database | PostgreSQL (Neon) with `django-tenants` (schema-based multi-tenancy) |
| Gateway | Kong (DB-less): JWT auth, CORS, rate limiting |
| Microservices | FastAPI, Pydantic v2, Uvicorn (22 services, direct sequential execution) |
| Queue | Celery + Redis (beat scheduler, `orchestration` queue), Apache Kafka (event streaming) |
| Storage | Google Cloud Storage (2 buckets: raw + curated) |
| Payments | Stripe (Basic $29 / Pro $79 / Enterprise $199) |
| Deployment | Railway (Docker), GitHub Actions CI/CD |

## Monorepo Layout

```
ai-brand-automator/           → Django backend (DRF, JWT, django-tenants)
ai-brand-automator-frontend/  → Next.js 15 + React 19 + TypeScript + Tailwind v4
pipeline-orchestrator-svc/    → FastAPI — sequential pipeline execution (port 8010)
discovery-agent-svc/          → FastAPI — web research via Tavily (port 8020)
intelligence-agent-svc/       → FastAPI — ISO 10668 brand valuation (port 8030)
chat-titling-worker/          → FastAPI — auto-titles chat sessions (port 8040)
content-agent-service/        → FastAPI — SEO/AEO/GEO blog authoring (port 8050)
social-agent-service/         → FastAPI — social media promotion (port 8060)
rag-uploader-agent-service/   → FastAPI — RAG document archival (port 8070)
brand-equity-calculator-svc/  → FastAPI — public brand equity calc, Anthropic Claude (port 8090)
market-research-agent-svc/    → FastAPI — Market sizing, TAM/SAM/SOM (port 8021)
competitor-intel-agent-svc/   → FastAPI — Competitor profiling, SWOT (port 8022)
audience-persona-agent-svc/   → FastAPI — Audience persona profiling (port 8023)
trend-cultural-agent-svc/     → FastAPI — Trend monitoring, cultural insights (port 8024)
voc-agent-svc/                → FastAPI — Voice of Customer analysis (port 8025)
brand-positioning-agent-svc/  → FastAPI — WF2 brand positioning (port 8031)
brand-architecture-agent-svc/ → FastAPI — WF2 brand architecture (port 8032)
brand-personality-agent-svc/  → FastAPI — WF2 brand personality (port 8033)
brand-naming-agent-svc/       → FastAPI — WF2 naming & tagline (port 8034)
brand-story-agent-svc/        → FastAPI — WF2 brand story & narrative (port 8035)
campaign-architecture-agent-svc/ → FastAPI — WF3 campaign architecture (port 8041)
creative-generation-agent-svc/ → FastAPI — WF3 creative generation (port 8042)
ad-publishing-agent-svc/      → FastAPI — WF3 ad publishing, Meta Ads API, human approval gate (port 8043)
odoo-mcp-server-svc/          → FastAPI — Odoo ERP MCP bridge, RBAC + 101 tools (port 8095)
odoo-worker-agent-svc/        → FastAPI — Multi-persona Odoo worker (port 8100)
vendor/odoo/community/        → Git submodule — Odoo Community Edition 19.0
deployment/                   → Docker Compose, Kong config, Railway/k8s manifests
docs/                         → Architecture docs, plans, guides
```

## Non-Negotiable Rules

### Backend

1. **Format**: Black (line-length 88) + Flake8. CI enforces `black --check .` and `flake8 .`
2. **Env vars**: Always `decouple.config()` with defaults and type casts — NEVER `os.environ`
3. **Multi-tenancy**: Always use `getattr(request, 'tenant', None)` — NEVER `request.tenant` directly
4. **Migrations**: Use `migrate_schemas --shared --noinput` — NEVER plain `migrate`
5. **ViewSets**: Use `select_related` on FK querysets, `get_serializer_class()` for action-specific serializers, `perform_create()` for tenant attachment
6. **MIME types**: Always use explicit MIME maps — NEVER rely solely on `mimetypes.guess_type()` (Docker containers lack `/etc/mime.types`)
7. **Encrypted tokens**: OAuth tokens live in `_access_token` columns, exposed via `@property` using `encrypt_token()`/`decrypt_token()`
8. **Pipeline apps** (`data_ingestion/`, `media_curation/`, `rag_index/`): Use **Hexagonal Architecture** — Pydantic domain models (NOT Django ORM), ABC ports, concrete adapters
9. **Test client**: Always set `client.defaults["SERVER_NAME"] = "localhost"` for tenant middleware
10. **Orchestration callbacks**: Use `transaction.atomic()` + `select_for_update()` around job state updates to prevent concurrent callback races
11. **SSRF prevention**: Validate external URLs in pipeline manifests against `ALLOWED_URL_PREFIXES` allowlist
12. **JSON size limits**: Validate `progress` and `result_data` payloads ≤ 1 MB in `CallbackSerializer`

### Frontend

1. **API calls**: Always use `apiClient` from `@/lib/api` — NEVER raw `fetch()`
2. **Route protection**: Always call `useAuth()` hook in protected pages

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/naveenah) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
