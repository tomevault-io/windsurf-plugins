---
trigger: always_on
description: Vulture is an application that loads source code from a local folder or git repository and inspects it for compliance against:
---

# Vulture - Compliance Audit Platform

## Project Overview

Vulture is an application that loads source code from a local folder or git repository and inspects it for compliance against:

1. **Chaos Engineering principles**
2. **OWASP guidelines**
3. **SOC2** (configurable down to specific compliance clauses)

Each audit option is further configurable based on complexity. For SOC2, users select specific compliance clauses to audit against. The system is built to be extensible for other types of compliance and audits.

AI agents for each audit type are launched independently. Each agent has precisely defined skills (documented in SKILLS.md) and uses the OpenAI Agents SDK (https://github.com/openai/openai-agents-python) with support for OpenAI, Claude, and Gemini models.

## Architecture

```
Frontend (React SPA + Vite) → SSE/REST → Go Backend → HTTP/SSE → Python Agent Services
                                              ↓
                                     PostgreSQL + pgvector
```

- **Go Backend** (`backend/`): Orchestrator. Receives audit requests, manages sources (git clone / local path), dispatches to Python agents concurrently, aggregates SSE streams, serves structured SSE events to frontend. PostgreSQL (pgvector) for production, SQLite fallback for local dev.
- **Python Agents** (`agents/`): Each audit type (chaos, owasp, soc2) is a separate FastAPI microservice using OpenAI Agents SDK + LiteLLM. Shared library in `agents/shared/`.
- **Frontend** (`frontend/`): React SPA (Vite) + Tailwind + react-i18next. Plain React with native EventSource for SSE streaming. Look and feel must be elegant like https://agentation.dev — intuitive, simple, elegant. Warm cream theme, compact sidebar, terminal-style agent output.
- **CLI** (`cli/`): Go CLI binary for headless audit execution (`vulture scan`, `vulture watch`, `vulture list`).
- **Deployment**: `docker compose` with all services (PostgreSQL, backend, 9 agents, frontend).

### Deployment Modes

Same binaries and Docker images serve all modes. Mode selection is via env vars only.

| Mode | Who runs it | Command | Notes |
|------|-------------|---------|-------|
| A: Dev-local | Developer laptop | `docker compose up` | SQLite or local Postgres; `VULTURE_LOCAL_MODE=true`; no new env vars required |
| B: Centralized server | Ops VM | `docker compose up -d` + Neon DSN + `VULTURE_API_KEYS_ENABLED=true` | See `docs/guides/central_server_deployment.md` (feature 0031) |
| C: Read-only viewer VM | Ops VM | `docker compose -f docker-compose.readonly.yml up -d` | Optional; set `VULTURE_READONLY=true`. See feature 0030 + `docs/guides/neon_deployment.md` |
| D: CI client | GitHub Actions etc. | `vulture scan <git-url> --api-key X --server Y --wait` | See `docs/guides/ci_integration.md` (feature 0031) |
| E: Native install | Single-user laptop, no Docker | `curl -fsSL https://raw.githubusercontent.com/bobinson/vulture/main/install.sh \| sh` | One-shot nuclei-style installer; SQLite + bundled python; see `docs/guides/native_installation.md` (feature 0044) |

Mode A is the default when you clone the repo. No new env vars are required; all centralized features are opt-in.

## Directory Structure

```
vulture/
  backend/               # Go 1.24+ backend
    cmd/vulture/         # Entry point (serve, local_start, status, scan, version)
    internal/
      handler/           # HTTP handlers (audit, source, stream, auth, memory, agent, filesystem, health)
      service/           # Business logic (audit, source, stream, agent_proxy, memory, auth)
      repository/        # Data access (postgres_repo, sqlite_repo, *_memory_repo, user_repo, mocks)
      model/             # Data structures (audit, finding, source, user, agent, event, memory)
      server/            # HTTP server setup, middleware (CORS, logging, auth), request_id
      config/            # Environment configuration loading
      agui/              # SSE encoder & agent-to-agui translator
      embedding/         # Vector embedding client (OpenAI/Ollama compatible)
      localdev/          # Local dev launcher (detect, process management)
    pkg/
      gitutil/           # Git clone utilities
      fileutil/          # File tree walking
    internal/repository/migrations/  # SQL migrations + auto-runner (//go:embed; feature 0040)
    test/e2e/            # Go E2E tests
  agents/                # Python 3.12+
    shared/              # Common library
      shared/
        audit_runner.py  # Combined skill+LLM audit pipeline
        base_agent.py    # Agent factory
        llm/provider.py  # LiteLLM config, model resolution, context window detection
        tools/           # file_scanner, file_reader, file_lister, pattern_matcher, ast_parser, dependency_checker, git_history, memory_client
        transport/       # sse_app (FastAPI factory), event_emitter (SSE events)
        models/          # audit_request, audit_result, finding
      tests/             # Unit + E2E tests
    chaos_engineering/   # Chaos agent (skills: retry, circuit_breaker, timeout, fallback, blast_radius)
    owasp/               # OWASP agent (skills: injection, auth, crypto, misconfig, access_control)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bobinson/vulture](https://github.com/bobinson/vulture) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
