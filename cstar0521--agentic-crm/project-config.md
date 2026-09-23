---
trigger: always_on
description: A polyrepo **agentic AI CRM platform** with three main services and a separate infrastructure layer. Built as a production-ready foundation for AI-powered CRM applications.
---

# Agentic CRM — Claude Code Guide

## Project Overview

A polyrepo **agentic AI CRM platform** with three main services and a separate infrastructure layer. Built as a production-ready foundation for AI-powered CRM applications.

**Architecture at a glance:**
- `crm-ui` — React 18 + TypeScript frontend (port 3090)
- `crm-backend` — Spring Boot 3.3 / Java 21 API (port 8080)
- `crm-agents` — Python 3.12 multi-agent system using Google ADK (ports 8001–8006)
- `crm-infra` — Docker Compose for PostgreSQL, Redis, Qdrant, and full observability stack

---

## Quick Start

```bash
# 1. Start infrastructure (PostgreSQL, Redis, Qdrant, observability)
cd crm-infra && docker compose up -d && ./init-qdrant.sh

# 2. Start backend
cd crm-backend && docker compose up --build -d

# 3. Start agents
cd crm-agents && docker compose up --build -d

# 4. Start UI
cd crm-ui && docker compose up --build -d

# 5. Verify everything is up
./verify-foundation.sh
```

---

## Service Reference

### crm-ui (React + Vite + TypeScript)

**Tech stack:** React 18, TypeScript 5.5, Vite 5, Salt DS, Zustand, TailwindCSS, Highcharts

```bash
cd crm-ui
npm install          # Install deps
npm run dev          # Dev server (Vite HMR)
npm run build        # Production build
npm run type-check   # TypeScript check (tsc --noEmit)
npm run lint         # ESLint
npm run test         # Vitest unit tests
npm run test:coverage
```

**Key directories:**
- `src/components/chat/` — ChatWindow, MessageInput, MessageBubble, AgentStatusIndicator
- `src/a2ui/` — A2UIRenderer (safe component catalog), useA2UIStream hook
- `src/api/` — conversationApi, agentApi, streamingApi, sessionApi
- `src/store/` — Zustand global state
- `src/types/` — TypeScript types for A2UI, agents, sessions, conversations

---

### crm-backend (Spring Boot / Java 21)

**Tech stack:** Spring Boot 3.3, PostgreSQL 16, Redis 7, Qdrant gRPC, Flyway, Spring WebFlux, SpringDoc OpenAPI

```bash
cd crm-backend
mvn spring-boot:run       # Local dev
mvn test                  # Tests (uses Testcontainers)
mvn package -DskipTests   # Build JAR
docker compose up --build -d
```

**Key packages under `src/main/java/com/crm/backend/`:**
- `agent/` — `AgentGatewayController`, `AgentGatewayService` (A2A task delegation)
- `streaming/` — `SseController`, `StreamingEventService` (SSE emitter registry)
- `session/` — `SessionService` (Redis TTL, 30 min default)
- `conversation/` — `ConversationService` (append-only messages)
- `memory/` — Four services: Working (Redis), Semantic (Qdrant), Episodic (PostgreSQL), Procedural (PostgreSQL JSONB)
- `context/` — `ContextFabricService` (aggregates all memory types, Redis-cached 5 min)
- `compliance/` — `ComplianceService` (immutable audit trail)

**Database migrations:** `src/main/resources/db/migration/` (V1–V10 Flyway SQL files)

**Environment variables:**
```
POSTGRES_URL, POSTGRES_USER, POSTGRES_PASSWORD
REDIS_HOST, REDIS_PORT
QDRANT_HOST, QDRANT_PORT
ORCHESTRATOR_URL=http://crm-orchestrator:8001
MEMORY_AGENT_URL=http://crm-memory-agent:8002
CONTEXT_AGENT_URL=http://crm-context-agent:8003
GUARDRAILS_URL=http://crm-guardrails:8004
OTEL_EXPORTER_OTLP_ENDPOINT
ENVIRONMENT
```

---

### crm-agents (Python / Google ADK)

**Tech stack:** Python 3.12, Google ADK, FastAPI, Gemini (Vertex AI or API key), Qdrant, Redis, Presidio (PII), spaCy, Perplexity API

**Package manager:** `uv` (not pip/poetry)

```bash
cd crm-agents
uv sync                                                   # Install deps
uv run uvicorn orchestrator.agent:app --reload --port 8001  # Single agent dev
uv run pytest tests/ -v                                   # Run tests
docker compose up --build -d                              # All 6 agents
```

**Agent services:**

| Agent | Port | Purpose |
|-------|------|---------|
| `orchestrator` | 8001 | Main intent routing + Gemini generation |
| `memory_agent` | 8002 | 4-type memory CRUD + semantic search |
| `context_agent` | 8003 | Context fabric aggregation |
| `guardrails` | 8004 | Input/output safety (PII, injection, toxicity) |
| `web_search_agent` | 8005 | Google Custom Search / DuckDuckGo fallback |
| `news_research_agent` | 8006 | Perplexity deep research |

**Shared code:** `shared/` — A2A protocol models, backend client, Gemini client, telemetry setup

**Environment variables:**
```
GOOGLE_API_KEY           # Gemini access
GEMINI_MODEL             # Default: gemini-2.5-flash
PERPLEXITY_API_KEY
GOOGLE_CSE_ID            # Optional, falls back to DuckDuckGo
BACKEND_URL=http://crm-backend:8080
REDIS_HOST, REDIS_PORT
QDRANT_HOST, QDRANT_PORT
OTEL_EXPORTER_OTLP_ENDPOINT
```

---

### crm-infra (Infrastructure)

```bash
cd crm-infra
docker compose up -d    # Start all infra services
./init-qdrant.sh        # Initialize Qdrant collections (run once)
```

| Service | Port | Purpose |
|---------|------|---------|
| PostgreSQL 16 | 5432 | Primary persistence |
| Redis 7 | 6379 | Cache + working memory |
| Qdrant 1.9.6 | 6333/6334 | Vector search (REST/gRPC) |
| Jaeger | 16686 | Distributed tracing UI |
| Prometheus | 9090 | Metrics |
| Loki | 3100 | Log aggregation |
| Grafana | 3000 | Dashboards (default: admin/admin) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cstar0521/agentic-crm](https://github.com/cstar0521/agentic-crm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
