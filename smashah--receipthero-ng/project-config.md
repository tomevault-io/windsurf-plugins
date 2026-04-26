---
trigger: always_on
description: AI-powered receipt management companion for Paperless-ngx. Automatically extracts, organizes, and converts receipts using Together AI's Llama 4 Maverick vision model.
---

# ReceiptHero

AI-powered receipt management companion for Paperless-ngx. Automatically extracts, organizes, and converts receipts using Together AI's Llama 4 Maverick vision model.

## Architecture

Turborepo monorepo with separate frontend, API, and worker processes sharing a core business logic layer:

```
receipthero-ng/
├── apps/
│   ├── api/           # Hono REST API (Bun runtime, port 3001)
│   ├── webapp/        # TanStack Start frontend (React 19, port 3002)
│   └── worker/        # Background polling & processing (single file)
├── packages/
│   ├── core/          # Core services & business logic (~31k tokens)
│   └── shared/        # Shared types, schemas, Zod validators
└── _legacy/           # Old Next.js implementation (DO NOT USE)
```

## Intent Layer

**Before modifying code in subdirectories with AGENTS.md, read that file first** to understand local patterns and invariants.

Future child nodes (not yet created):
- **Core Services**: `packages/core/AGENTS.md` - Business logic, external integrations, database
- **Frontend**: `apps/webapp/AGENTS.md` - TanStack Router patterns, UI components, real-time updates

## System Flow

```
1. Worker polls Paperless-ngx every N seconds for documents tagged "receipt"
2. Downloads document thumbnail (PDF first page → image)
3. Calls Together AI vision model to extract structured receipt data
4. (Optional) Converts currency using fawazahmed0 exchange API
5. Updates Paperless document with:
   - New title: "{Vendor} - {Amount} {Currency}"
   - Correspondent (vendor)
   - Custom fields (full receipt JSON)
   - Tags ("ai-processed")
6. Logs results to SQLite for dashboard display
```

## Runtime Architecture

### Three Separate Processes

**API Server** (`apps/api`)
- Hono-based REST API serving `/api/*` endpoints
- WebSocket server for live log streaming (`/api/ws`)
- Stateless - reads/writes SQLite and config.json
- Does NOT process receipts (that's the worker's job)

**Worker** (`apps/worker`)
- Single background process with infinite loop
- Polls `worker_state` table to check if paused or manually triggered
- Calls `runAutomation()` from core to scan Paperless and process receipts
- Acquires lock before running to prevent duplicate scans
- Graceful shutdown handling (SIGTERM/SIGINT)

**Webapp** (`apps/webapp`)
- TanStack Start SPA (Vite + React 19)
- Real-time dashboard consuming API endpoints
- WebSocket connection for live log streaming
- Fully client-side (no SSR in production)

### Communication Patterns

- **API ↔ Core**: Direct function calls (same process)
- **Worker ↔ Core**: Direct function calls (same process)
- **API ↔ Worker**: Shared SQLite database (`worker_state` table for control)
- **Webapp ↔ API**: REST + WebSocket
- **All ↔ Paperless**: HTTP via `PaperlessClient` in core
- **All ↔ Together AI**: HTTP via `Together` SDK in core

## Key Technologies

| Layer | Stack |
|-------|-------|
| Runtime | Bun (API + Worker), Vite (Webapp) |
| Frontend | React 19, TanStack Router/Query, Base UI, Tailwind CSS 4 |
| Backend | Hono, Zod validation, Drizzle ORM |
| Database | SQLite (better-sqlite3) |
| AI/ML | Together AI (Llama 4 Maverick vision model) |
| External APIs | Paperless-ngx (document management), fawazahmed0 (currency rates) |

## Database Schema

Single SQLite database (`receipthero.db`) with 5 tables:

1. **`retry_queue`**: Failed documents awaiting retry (exponential backoff)
2. **`processing_logs`**: Full audit trail of all document processing attempts
3. **`logs`**: Application logs (worker, api, core) for debugging
4. **`worker_state`**: Single-row table for pause/resume/trigger control
5. **`skipped_documents`**: Documents permanently skipped (e.g., no receipt data)

See `packages/core/src/db/schema.ts` for full DDL.

## Configuration

**Two configuration sources:**

1. **Environment variables** (`.env`)
   - `DATABASE_PATH`: SQLite file location (default: `./receipthero.db`)
   - `CONFIG_PATH`: JSON config location (default: `./config.json`)
   - `PORT`: API server port (default: 3001)
   - Optional: `PAPERLESS_HOST`, `PAPERLESS_API_KEY`, `TOGETHER_API_KEY` (can be set via UI)

2. **`config.json`** (runtime configuration, editable via webapp)
   - Paperless-ngx connection details
   - Together AI API key
   - Processing settings (scan interval, tag names, currency conversion)
   - Use document type detection instead of tag-based filtering

**Priority**: `config.json` values override env vars (except paths/port).

Load config via `loadConfig()` from `@sm-rn/core` - never read files directly.

## Core Services Overview

All services in `packages/core/src/services/`:

| Service | Responsibility |
|---------|----------------|
| `paperless.ts` | Paperless-ngx API client (fetch documents, update metadata, manage tags/correspondents) |
| `ocr.ts` | Together AI vision model wrapper (extract receipt data from images) |
| `together-client.ts` | Together AI SDK singleton with rate limiting (Upstash Redis) |
| `fawazahmed0.ts` | Currency conversion (primary - dual CDN fallback) |
| `ecb.ts` | European Central Bank rates (backup - not currently used) |
| `retry-queue.ts` | Exponential backoff retry logic for failed documents |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [smashah/receipthero-ng](https://github.com/smashah/receipthero-ng) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
