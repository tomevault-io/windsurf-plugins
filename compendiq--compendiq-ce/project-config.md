---
trigger: always_on
description: This file provides guidance to Claude Code when working with this repository. AGENTS.md mirrors these rules for other AI tools.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository. AGENTS.md mirrors these rules for other AI tools.

## Project Overview

**Compendiq** — AI-powered knowledge base management web app that integrates with Confluence Data Center (on-premises) and supports multiple LLM providers (Ollama, OpenAI-compatible APIs) for article improvement, generation, summarization, and RAG-powered Q&A. Multi-user: each user configures their own Confluence PAT and space selections. Monorepo: `backend/` (Fastify 5 + PostgreSQL + Redis) and `frontend/` (React 19 + Vite).

See `@docs/ARCHITECTURE-DECISIONS.md` for all ADRs. See `@docs/ACTION-PLAN.md` for the implementation plan.

## Mandatory Rules

1. **Tests required** — Every change needs tests. Backend: `backend/src/**/*.test.ts`, Frontend: `frontend/src/**/*.test.{ts,tsx}`, E2E: `e2e/*.spec.ts`. Both use Vitest; frontend uses jsdom + `@testing-library/react`. Never use `--no-verify`.
2. **Never push to `main`** — Branch from `dev` as `feature/<desc>`. PRs go `feature/* -> dev`. Only `dev -> main` merges target `main`.
3. **Never commit secrets** — No `.env`, API keys, PATs, passwords, or credentials.
4. **Ask before assuming** — If ambiguous, ask for clarification before proceeding.
5. **Follow the ADRs** — All architectural decisions are in `docs/ARCHITECTURE-DECISIONS.md`. Do not deviate without discussion.

## Build Commands

```bash
npm install                # Install all (all workspaces)
npm run dev                # Dev server (backend + frontend)
npm run build              # Build everything
npm run lint               # Lint
npm run typecheck          # Type check
npm test                   # All tests
npm run test -w backend    # Backend only
npm run test -w frontend   # Frontend only
# Single file: cd backend && npx vitest run src/path/file.test.ts
# Backend tests use real PostgreSQL (POSTGRES_TEST_URL env var, default: localhost:5433)
# E2E: npx playwright test (requires running backend + frontend)
# Docker: docker compose -f docker/docker-compose.yml up -d
```

## Architecture

Flat monorepo with domain-based backend structure and shared contracts (ADR-001, ADR-008):

```
compendiq/
├── backend/src/
│   ├── core/                        # Shared infrastructure (no domain imports)
│   │   ├── enterprise/              # Enterprise plugin loader (types, noop, loader, features)
│   │   ├── db/postgres.ts           # Connection pool + migration runner
│   │   ├── db/migrations/           # Sequential SQL files (001-049)
│   │   ├── plugins/                 # Fastify plugins (auth, correlation-id, redis)
│   │   ├── services/                # Cross-cutting: redis-cache, audit-service, error-tracker,
│   │   │                            #   content-converter, circuit-breaker, image-references,
│   │   │                            #   rbac-service, notification-service,
│   │   │                            #   pdf-service, admin-settings-service, version-snapshot
│   │   └── utils/                   # crypto, logger, sanitize-llm-input, ssrf-guard, tls-config, llm-config
│   ├── domains/
│   │   ├── confluence/services/     # confluence-client, sync-service, attachment-handler,
│   │   │                            #   subpage-context, sync-overview-service
│   │   ├── llm/services/            # ollama-service, ollama-provider, openai-service,
│   │   │                            #   llm-provider, embedding-service, rag-service, llm-cache
│   │   └── knowledge/services/      # auto-tagger, quality-worker, summary-worker,
│   │                                #   version-tracker, duplicate-detector
│   ├── routes/
│   │   ├── foundation/              # health, auth, settings, admin, rbac, notifications
│   │   ├── confluence/              # spaces, sync, attachments
│   │   ├── llm/                     # llm-chat (SSE streaming), llm-conversations,
│   │   │                            #   llm-embeddings, llm-models, llm-admin, llm-pdf
│   │   └── knowledge/               # pages-crud, pages-versions, pages-tags,
│   │                                #   pages-embeddings, pages-duplicates, pinned-pages,
│   │                                #   analytics, knowledge-admin, templates, comments,
│   │                                #   content-analytics, verification, knowledge-requests,
│   │                                #   search, pages-export, pages-import, local-spaces
│   ├── app.ts                       # Fastify app builder + route registration
│   └── index.ts                     # Entry point + workers
├── frontend/src/
│   ├── features/         # Domain-grouped UI (admin, ai, analytics, auth, dashboard,
│   │                     #   graph, knowledge-requests, pages, search, settings, spaces, templates)
│   ├── shared/           # Reusable components, hooks, lib
│   │   ├── enterprise/   # Enterprise plugin loader (context, types, loader, hook)
│   │   └── components/   # Categorized: layout/, article/, diagrams/, badges/, feedback/, effects/
│   ├── stores/           # Zustand stores (auth, theme, ui, article-view, command-palette, keyboard-shortcuts)
│   └── providers/        # Context providers (Query, Auth, Router)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Compendiq) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-15 -->
