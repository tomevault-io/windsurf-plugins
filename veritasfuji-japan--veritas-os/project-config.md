---
trigger: always_on
description: - **Name**: VERITAS OS v2.0.0
---

# VERITAS OS — AI Coding Agent Instructions

## 1. Project Identity

- **Name**: VERITAS OS v2.0.0
- **Purpose**: Auditable Decision OS for LLM Agents (Proto-AGI Skeleton)
- **Repository**: https://github.com/veritasfuji-japan/veritas_os
- **Author**: Takeshi Fujishita
- **License**: Multi-license (Core = Proprietary EULA, Interface = MIT)
- **Status**: Beta (Pre-release) — Technical DD Score: 84/100 (A-)

## 2. Tech Stack

### Backend (Python)
- **Language**: Python 3.11+ (target: 3.12.12)
- **Framework**: FastAPI 0.121.0 + Uvicorn
- **Data Validation**: Pydantic v2 (2.8.2)
- **LLM Client**: OpenAI SDK (1.51.0), httpx (0.27.2)
- **Serialization**: orjson
- **Build**: setuptools + pyproject.toml
- **Linter**: Ruff (target py311, select: E/F/W/B)
- **Tests**: pytest 8.3.5 + pytest-asyncio + pytest-cov
- **CI Coverage Gate**: ≥ 85% (`--cov-fail-under=85`)
- **Task Runner**: Makefile with `uv` (astral)

### Frontend (TypeScript)
- **Framework**: Next.js 16 (App Router, React 18)
- **Language**: TypeScript 5.7
- **Styling**: Tailwind CSS 3.4 + CVA (class-variance-authority)
- **Package Manager**: pnpm (workspace)
- **Testing**: Vitest + Testing Library (unit), Playwright + axe-core (E2E)
- **i18n**: Custom React Context (ja default, en)
- **Design System**: `@veritas/design-system` (packages/design-system/)
- **Shared Types**: `@veritas/types` (packages/types/) with runtime type guards

### Infrastructure
- Docker + Docker Compose (backend:8000 + frontend:3000)
- GHCR: `ghcr.io/veritasfuji-japan/veritas_os:latest`
- GitHub Actions CI (Python 3.11/3.12 matrix, CodeQL, SBOM)

## 3. Repository Structure (Critical Paths)

```
veritas_os/                     ← Monorepo root
├── veritas_os/                 ← Python backend
│   ├── api/                    ← FastAPI server, routes, schemas, governance
│   │   ├── server.py           ← FastAPI app (40+ endpoints)
│   │   ├── routes_decide.py    ← /v1/decide & replay
│   │   ├── routes_trust.py     ← TrustLog & audit
│   │   ├── routes_memory.py    ← Memory CRUD
│   │   ├── routes_governance.py← Governance & policy
│   │   ├── routes_system.py    ← Health, metrics, SSE, halt
│   │   ├── schemas.py          ← Pydantic v2 request/response
│   │   └── governance.py       ← Policy mgmt, 4-eyes approval, RBAC/ABAC
│   ├── core/                   ← Decision engine
│   │   ├── kernel.py           ← Decision computation
│   │   ├── pipeline/           ← 20+ stage orchestrator (package)
│   │   ├── fuji/               ← FUJI safety gate (package)
│   │   ├── memory/             ← MemoryOS (package)
│   │   ├── continuation_runtime/ ← Phase-1 observe/shadow
│   │   ├── value_core.py       ← Value alignment + EMA
│   │   ├── world.py            ← WorldModel
│   │   ├── llm_client.py       ← Multi-provider LLM gateway
│   │   ├── debate.py           ← Multi-viewpoint debate
│   │   ├── critique.py         ← Self-critique
│   │   ├── planner.py          ← Action planning
│   │   └── sanitize.py         ← PII masking
│   ├── policy/                 ← Policy compiler, signing, runtime adapter
│   ├── logging/                ← TrustLog, encryption, rotation
│   ├── audit/                  ← Ed25519 signed audit
│   ├── compliance/             ← EU AI Act reports
│   ├── security/               ← SHA-256, Ed25519
│   ├── replay/                 ← Deterministic replay engine
│   ├── observability/          ← OpenTelemetry
│   ├── storage/                ← Pluggable backends (JSONL, PostgreSQL)
│   ├── tools/                  ← Web search, GitHub search
│   ├── prompts/                ← LLM prompt templates
│   └── tests/                  ← 5600+ Python tests
├── frontend/                   ← Next.js 16 Mission Control
│   ├── app/                    ← Pages (/, /console, /audit, /governance, /risk)
│   ├── components/             ← Shared React components
│   ├── features/console/       ← Decision Console feature
│   ├── lib/                    ← API client, validators, utilities
│   ├── locales/                ← i18n files
│   └── e2e/                    ← Playwright E2E tests
├── packages/
│   ├── types/                  ← Shared TS types + runtime validators
│   └── design-system/          ← Card, Button, AppShell
├── spec/                       ← OpenAPI spec (MIT)
├── sdk/                        ← SDK interface (MIT)
├── cli/                        ← CLI interface (MIT)
├── policies/                   ← Policy templates
├── scripts/                    ← Architecture/quality/security checks
├── openapi.yaml                ← OpenAPI 3.x
├── pyproject.toml              ← Python config
├── Makefile                    ← Dev/test commands
└── docker-compose.yml          ← Full-stack orchestration
```

## 4. Architecture Principles (MUST Follow)

### 4.1 Responsibility Boundaries (Enforced in CI)

These boundaries are verified by `scripts/architecture/check_responsibility_boundaries.py`:

| Component    | Owns                                          | Must NOT absorb                                |
|-------------|-----------------------------------------------|-----------------------------------------------|
| **Planner** | Planning structure, action-plan generation    | Kernel orchestration, FUJI policy, Memory I/O |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [veritasfuji-japan/veritas_os](https://github.com/veritasfuji-japan/veritas_os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
