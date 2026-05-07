---
trigger: always_on
description: This workspace contains two repositories:
---

# Copilot Instructions for Open Agent School

This workspace contains two repositories:
- **OpenAgentSchool** - React frontend (this repo)
- **openagent-backend** - Python microservices backend (sibling folder at `C:\code\openagent-backend`)

## Frontend (OpenAgentSchool)

### Build, Test, and Lint

```bash
# Install dependencies
npm install

# Development server (http://localhost:5000)
npm run dev

# Build (TypeScript check + Vite production build)
npm run build

# Lint
npm run lint

# Run all tests
npm run test

# Run a single test file
npx vitest run tests/dataAutonomyPatterns.test.ts

# Run tests matching a pattern
npx vitest run -t "pattern name"

# Watch mode
npm run test:watch

# Skills-driven eval guardrail (Agentic Eval)
npm run test:evaluation
```

### Skills Optimization Run Loop (Applied)

Use this fast loop before broader test/build runs:
1. **Agentic Eval**: `npm run test:evaluation` to verify every pattern has a usable evaluation profile.
2. **Governance**: `npx vitest run tests/unit/accessControlPolicy.test.ts tests/studyModeContentGuardrails.test.ts`.
3. **Web App Smoke**: `npx vitest run tests/smoke/app-smoke.test.tsx`.

## Backend (openagent-backend)

Three microservices with independent Docker Compose setups:

| Service | Port | Purpose |
|---------|------|---------|
| **core-api** | 8000 | User auth, community, quizzes, progress tracking |
| **agent-orchestrator** | 8002 | Multi-agent AI, critical thinking, educational frameworks |
| **knowledge-service** | 8003 | Document processing, semantic search, RAG |

### Running Services

```powershell
# Start all services (from openagent-backend directory)
cd C:\code\openagent-backend\core-api
docker compose up -d --build

cd C:\code\openagent-backend\agent-orchestrator
docker compose up -d --build

cd C:\code\openagent-backend\knowledge-service
docker compose up -d --build

# Health checks
curl http://localhost:8000/health
curl http://localhost:8002/api/v1/health/live
curl http://localhost:8003/health
```

### Running Without Docker

```powershell
# Core API
cd C:\code\openagent-backend\core-api
pip install -r requirements.txt
cp .env.example .env
python run.py

# Agent Orchestrator (requires Redis)
cd C:\code\openagent-backend\agent-orchestrator
pip install -r requirements.txt
docker run -d -p 6379:6379 redis:7-alpine
uvicorn main:app --reload --port 8002
```

### Backend Tests

```bash
# Core API
cd C:\code\openagent-backend\core-api
pytest -q

# Agent Orchestrator
cd C:\code\openagent-backend\agent-orchestrator
pytest

# Knowledge Service
cd C:\code\openagent-backend\knowledge-service
pytest --cov
```

### Frontend-Backend Integration

Create `.env.local` in OpenAgentSchool:
```env
VITE_CORE_API_URL=http://localhost:8000
VITE_ORCHESTRATOR_SERVICE_URL=http://localhost:8002
VITE_KNOWLEDGE_SERVICE_URL=http://localhost:8003
```

## Architecture

### Frontend (OpenAgentSchool)

**React 18 + TypeScript + Vite** single-page learning platform. Works standalone; backend enables advanced features.

**Key Directories:**
- `src/components/concepts/` - Individual concept pages (AgentArchitectureConcept.tsx, MCPConcept.tsx, etc.)
- `src/components/patterns/` - Agent pattern implementations
- `src/components/visualization/` - D3 visualizations including the Learning Atlas taxonomy tree
- `src/components/study-mode/` - Socratic learning and evaluation logic
- `src/components/ui/` - Radix UI-based design system components
- `tests/` - Vitest tests with jsdom environment

**Routing:** Defined in `src/App.tsx` using react-router-dom v7. Components are lazy-loaded via `React.lazy()`.

**Data Flow:**
- Concept definitions live inline in component files (e.g., `ConceptsHub.tsx`)
- Stable concept IDs (kebab-case like `agent-learning`, `fine-tuning`) are used for cross-component linking
- State is component-local or React Query for API calls

### Backend (openagent-backend)

**Microservices architecture** with polyglot persistence:

| Service | Database | Vector Store |
|---------|----------|--------------|
| core-api | DuckDB (dev) / Cosmos DB (prod) | Optional ChromaDB |
| agent-orchestrator | SQLite (dev) / PostgreSQL (prod) | N/A |
| knowledge-service | PostgreSQL | ChromaDB / Qdrant |

**Service Communication:**
- REST APIs with OpenAPI docs at `/docs` for each service
- No service mesh; direct HTTP calls between services
- JWT auth from core-api, API keys for service-to-service

**Key Backend Directories:**
- `core-api/app/` - User management, community, quizzes, progress
- `agent-orchestrator/app/` - Multi-agent AI, critical thinking frameworks (Bloom's, Paul-Elder)
- `knowledge-service/app/` - Document processing, semantic search, RAG pipeline

## Conventions

### Code Style

- **Functional React components** with explicit prop types
- **Tailwind utility-first** styling; group as: layout → spacing → color → motion
- **Import order**: React/libs → hooks/context → components → styles/static
- **Path alias**: `@/` maps to `src/`
- No CSS-in-JS; stick to Tailwind

### TypeScript

- Current build uses `--noCheck` shortcut; avoid introducing strict errors
- Unused vars: prefix with underscore (`_unused`) to avoid lint warnings

### Commits

- Format: `<scope>: <brief>` (e.g., `concepts: add fine-tuning to journey map`)
- Keep commits small and logically scoped


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bhakthan/OpenAgentSchool](https://github.com/bhakthan/OpenAgentSchool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
