---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) / Codex when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) / Codex when working with code in this repository.

## Project Overview

StructureClaw is an AI-assisted structural engineering workspace. It takes natural-language structural descriptions through a conversational UI and runs them through a pipeline: draft model → validate → analyze → code-check → report. The system is bilingual (`en` / `zh`); all new user-visible text must ship in both locales.

## Architecture

```text
frontend (Next.js 14)  →  backend (Fastify + Prisma)  →  analysis engine (e.g. Python analysis runtime (OpenSees))
```

- **Backend** (`backend/`): Fastify API with Prisma ORM (SQLite default). Route handlers in `src/api`, orchestration/domain logic in `src/services`, shared helpers in `src/utils`.
- **Frontend** (`frontend/`): Next.js 14 app-router. Routes in `src/app`, reusable UI in `src/components`, client state (Zustand) and i18n in `src/lib`.
- **Agent runtime** (`backend/src/agent-runtime/`): Capability-driven orchestration — base model (chat) → skill layer (engineering advisor) → tool layer (executable agent). 14 skill domains live under `backend/src/agent-skills/` (structure-type, analysis, code-check, data-input, design, drawing, general, load-boundary, material, report-export, result-postprocess, section, validation, visualization).
- **Analysis execution** (`backend/src/agent-skills/analysis/`): Spawns a Python subprocess using OpenSees for structural analysis. Python runtime lives in `backend/.venv/`.
- **CLI** (`scripts/cli/`): The `./sclaw` and `./sclaw_cn` commands wrap all startup, health-check, and lifecycle operations. Always prefer `./sclaw ...` over calling script paths directly.
- **Tests** (`tests/`): Regression and contract validation via `node tests/runner.mjs`.

## Build & Run

**Environment setup (first time):**
```bash
./sclaw doctor          # checks deps, installs missing, sets up backend/.venv
./sclaw start           # starts backend + frontend
./sclaw status          # verify both services running
./sclaw logs            # tail logs
./sclaw stop            # graceful shutdown
```

**China mirror variant:** Replace `./sclaw` with `./sclaw_cn` for mirror defaults (npmmirror, tsinghua PyPI, Docker mirror).

**Backend only:**
```bash
npm run build --prefix backend          # TypeScript compile
npm run lint --prefix backend           # ESLint
npm test --prefix backend -- --runInBand  # Jest (internally runs db:generate + build first)
npm run db:generate --prefix backend    # Regenerate Prisma client (required before build after schema changes)
npm run db:push --prefix backend        # Sync Prisma schema to SQLite (no migration files)
```

**Frontend only:**
```bash
npm run build --prefix frontend         # Next.js production build
npm run type-check --prefix frontend    # tsc --noEmit
npm run test:run --prefix frontend      # Vitest (all tests)
npm run lint --prefix frontend          # ESLint
npm run test:e2e --prefix frontend      # Playwright E2E tests
```

**Running a single test:**
```bash
# Backend (Jest)
npm test --prefix backend -- --runInBand path/to/test.test.ts

# Frontend (Vitest)
npm run test:run --prefix frontend -- path/to/test.test.ts
```

**Regression and validation:**
```bash
node tests/runner.mjs analysis-regression                    # OpenSees analysis regression
node tests/runner.mjs check backend-regression               # Backend regression suite (build + lint + Jest + validations)
node tests/runner.mjs validate validate-agent-orchestration  # Agent orchestration contract
node tests/runner.mjs validate validate-chat-stream-contract # Chat stream contract
node tests/runner.mjs validate validate-analyze-contract     # Analyze endpoint contract
node tests/runner.mjs smoke-native                           # Full native install smoke (mirrors CI)
node tests/runner.mjs smoke-docker                           # Docker smoke (needs Docker)
node tests/runner.mjs llm-integration                        # LLM integration tests (needs LLM_API_KEY)
```

## Key Conventions

- **TypeScript**: strict mode, ES modules, 2-space indent, semicolons. Explicit types at API/store boundaries. Route handlers stay thin; logic goes into services.
- **Naming**: files = lowercase domain (`agent.ts`, `analysis.ts`), classes = `PascalCase`, functions/variables = `camelCase`, constants = `UPPER_SNAKE_CASE`.
- **Bilingual**: Every user-visible string must support `en` and `zh`. Route through the existing i18n system; never hardcode single-language strings.
- **Frontend**: Preserve Next.js app-router structure. Use existing Zustand store, i18n, and theme infrastructure. Prefer design-token styling over hardcoded light/dark values.
- **Python**: Follow existing style in analysis modules. Keep regression fixtures deterministic.
- **Immutability**: Prefer creating new objects over mutating existing ones.
- **Commits**: Conventional commit format (`feat(frontend): ...`, `fix(backend): ...`, `docs: ...`). Small logical commits with clear boundaries; don't bundle unrelated work.

## Module Boundaries

| Concern | Location |
|---------|----------|
| API routes | `backend/src/api/` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [structureclaw/structureclaw](https://github.com/structureclaw/structureclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
