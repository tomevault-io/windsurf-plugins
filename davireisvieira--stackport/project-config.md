---
trigger: always_on
description: StackPort is a universal AWS resource browser for local emulators (MiniStack, LocalStack, Moto, or any AWS-compatible endpoint). Python FastAPI backend with boto3, React frontend served as static files. Single Docker image.
---

# CLAUDE.md

## Project Overview

StackPort is a universal AWS resource browser for local emulators (MiniStack, LocalStack, Moto, or any AWS-compatible endpoint). Python FastAPI backend with boto3, React frontend served as static files. Single Docker image.

## Commands

```bash
# Backend
pip install -e .
AWS_ENDPOINT_URL=http://localhost:4566 stackport        # or: python -m backend.main

# Frontend dev
cd ui && npm install && npm run dev                      # dev server with proxy to :8080
cd ui && npm run build                                   # production build → ui/dist/

# Typecheck & lint
cd ui && npx tsc -b
cd ui && npx eslint .

# Docker
docker compose up                                        # StackPort + MiniStack
```

Requires a running AWS-compatible emulator (MiniStack on :4566 by default).

## Architecture

**Backend** (`backend/`):
- `main.py` — FastAPI app, CORS, static file mount for SPA, CLI entry point
- `config.py` — All settings from env vars (`AWS_ENDPOINT_URL`, `AWS_REGION`, `STACKPORT_PORT`, `STACKPORT_SERVICES`, `STACKPORT_ENDPOINTS`)
- `aws_client.py` — `get_client(service, endpoint_url)` with `@lru_cache(maxsize=256)` keyed on `(service, endpoint_url)`
- `routes/common.py` — `get_endpoint_url` FastAPI dependency resolves `?endpoint=` query param → endpoint URL
- `cache.py` — Thread-safe `TTLCache` singleton (dict + timestamps + threading.Lock)
- `routes/stats.py` — `GET /api/stats` — probes 35 services concurrently via ThreadPoolExecutor, cached 5s
- `routes/resources.py` — `GET /api/resources/{svc}` and `GET /api/resources/{svc}/{type}/{id}` — generic list/detail
- `routes/s3.py` — `GET /api/s3/buckets`, `/api/s3/buckets/{name}/objects`, `/api/s3/buckets/{name}/objects/{key}` — S3-specific with download support

**Schemas** (`backend/schemas/`):
- Pydantic request/response models, one module per service: `sqs.py`, `s3.py`, `dynamodb.py`, `tags.py`
- Route files import schemas — they never define `BaseModel` subclasses inline
- When adding a new service with write endpoints, create `backend/schemas/<service>.py`
- Uses `Field(alias="camelCase")` + `populate_by_name: True` for camelCase JSON ↔ snake_case Python

**Key registries in backend:**
- `SERVICE_REGISTRY` (stats.py) — maps service name → list of `(resource_type, boto3_service, method, response_key)` tuples. 35 services.
- `DESCRIBE_REGISTRY` (resources.py) — maps `(service, resource_type)` → boto3 describe call for detail views. 19 entries.
- `_METHOD_KWARGS` (stats.py) — extra params for APIs that require them (cognito `MaxResults`, wafv2 `Scope`).

**Frontend** (`ui/src/`):
- React 18 + Vite 5 + TypeScript + Tailwind CSS 3 + shadcn/ui (Radix-based)
- `main.tsx` — BrowserRouter basename `/`, TooltipProvider, Sonner Toaster
- `App.tsx` — Routes: `/` (Dashboard), `/resources/:service?` (ResourceBrowser), `*` → redirect
- `components/Layout.tsx` — Sidebar nav (Dashboard, Resources)
- `pages/Dashboard.tsx` — Service grid with status badges, resource counts, links to browser
- `pages/ResourceBrowser.tsx` — Service sidebar + resource table + detail Sheet. Renders `SERVICE_VIEWS[service]` when available, falls back to generic table.
- `components/service-views/index.ts` — Registry: `{ s3: S3Browser }`. Add new service UIs here.
- `components/service-views/S3Browser.tsx` — Full S3 file browser: bucket list, folder navigation, object detail, search, pagination, download.
- `lib/api.ts` — `API_BASE = '/api'`, fetch functions for all endpoints
- `lib/types.ts` — `ServiceStats`, `StatsResponse`
- `lib/service-icons.ts` — 35+ service → lucide icon mappings, fallback to `Server`
- `hooks/useFetch.ts` — Generic polling hook with toast on error

## Environment Variables

| Variable | Default | Purpose |
|---|---|---|
| `AWS_ENDPOINT_URL` | `http://localhost:4566` | AWS-compatible endpoint |
| `AWS_REGION` | `us-east-1` | Region for boto3 clients |
| `AWS_ACCESS_KEY_ID` | `test` | Credentials |
| `AWS_SECRET_ACCESS_KEY` | `test` | Credentials |
| `STACKPORT_PORT` | `8080` | HTTP port |
| `STACKPORT_S3_MAX_UPLOAD_MB` | `100` | Max S3 upload size per object (whole mebibytes; × 1024²) |
| `STACKPORT_SERVICES` | *(35 services)* | Comma-separated list to probe |
| `STACKPORT_PROBE_TIMEOUT` | `5` | Seconds before a service probe times out |
| `STACKPORT_CACHE_TTL` | `5` | Seconds to cache service stats |
| `STACKPORT_PROBE_WORKERS` | `10` | ThreadPoolExecutor max workers for concurrent probing |
| `LOG_LEVEL` | `INFO` | Python log level (DEBUG shows healthcheck logs) |

## Adding a New Service to the Backend

1. Add entries to `SERVICE_REGISTRY` in `backend/routes/stats.py` — `(resource_type, boto3_service, method, response_key)`
2. If the list API needs extra kwargs, add to `_METHOD_KWARGS` in `stats.py`
3. Add detail lookup to `DESCRIBE_REGISTRY` in `backend/routes/resources.py`
4. Add ID field names to `_ID_FIELDS` in `resources.py`
5. Add the service to `STACKPORT_SERVICES` default in `backend/config.py`

## Adding a Service-Specific UI View

For services that need richer UX than the generic resource table (like S3's file browser):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DaviReisVieira/stackport](https://github.com/DaviReisVieira/stackport) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
