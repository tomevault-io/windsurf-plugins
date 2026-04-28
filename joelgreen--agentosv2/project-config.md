---
trigger: always_on
description: AgentOS v2 is an AI agent orchestration platform. Ground-up Python rebuild.
---

# AgentOS v2

## Overview

AgentOS v2 is an AI agent orchestration platform. Ground-up Python rebuild.

- **Backend**: FastAPI + SQLAlchemy + asyncio — `src/engine/app/api/`
- **Frontend**: React + Vite + Tailwind — `src/engine/app/ui/`
- **Database**: PostgreSQL + pgvector — `src/engine/db/`
- **Conductor**: Claude Opus agent loop — primary interface (building Phase 1)
- **Cloud Execution**: AWS ECS Fargate (planned later)

## Documentation

**Product overview:**

| Doc | What it covers |
|-----|---------------|
| [docs/PRODUCT_MATRIX.md](docs/PRODUCT_MATRIX.md) | **Complete feature matrix** — every feature, status, design doc, milestone |

**Core docs** (read in order):

| Doc | What it covers |
|-----|---------------|
| [docs/MANIFESTO.md](docs/MANIFESTO.md) | WHY: business vision, product goals, competitive positioning, design principles |
| [docs/ROADMAP.md](docs/ROADMAP.md) | WHEN: milestones M1-M12, status, what's built, what's next |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | HOW: system design, package structure, DB schema, long-term 5-layer vision |
| [docs/conductor-architecture.md](docs/conductor-architecture.md) | Conductor agent: runtime, tools, services, memory, channels, workspaces |
| [docs/conductor-ui-architecture.md](docs/conductor-ui-architecture.md) | Conductor UI: layout, inline cards, expand pattern, component inventory |
| [docs/conductor-prompt-design.md](docs/conductor-prompt-design.md) | Conductor system prompt: identity, planning protocol, communication style, examples |
| [docs/ONBOARDING.md](docs/ONBOARDING.md) | Key files, running locally |

**Reference docs:**

| Doc | What it covers |
|-----|---------------|
| [docs/USE_CASES.md](docs/USE_CASES.md) | Use cases, three levels of automation, Slack hero example |
| [docs/DESIGN.md](docs/DESIGN.md) | Visual design system: colors, typography, spacing, components |
| [docs/PROVIDER_SPEC.md](docs/PROVIDER_SPEC.md) | Provider/capability protocol system, I/O types |
| [docs/WORKSPACE_PATTERNS.md](docs/WORKSPACE_PATTERNS.md) | How to ship work: parallel vs serial patterns, brief templates |
| [docs/VISION.md](docs/VISION.md) | Detailed Layer 2-5 designs (persistent agents, evolution, hierarchy, knowledge) |

**Feature designs:**

| Doc | What it covers |
|-----|---------------|
| [docs/auth-authority-design.md](docs/auth-authority-design.md) | Auth, multi-org, roles, authority profiles, escalation, approvals |
| [docs/integrations-design.md](docs/integrations-design.md) | Integrations: OAuth, API keys, MCP, databases, Composio, webhooks, multi-account |
| [docs/infrastructure-design.md](docs/infrastructure-design.md) | AWS infra: ECS Fargate, RDS, Redis, SQS, S3+CloudFront, CI/CD, monitoring |
| [docs/memory-rag-design.md](docs/memory-rag-design.md) | Memory/RAG: 4 memory types, multi-strategy retrieval, entity graph, outcome learning |
| [docs/documents-design.md](docs/documents-design.md) | Documents: git-native storage, BlockNote editor, indexing, agent tools, import |
| [docs/system-workflows-design.md](docs/system-workflows-design.md) | System workflows: agent capabilities as editable workflows, compiler to Python |
| [docs/data-views-design.md](docs/data-views-design.md) | Data views: compact cards + full tables per data type, shared DataTable component |
| [docs/data-explorer-design.md](docs/data-explorer-design.md) | NocoDB integration (DEFERRED — for advanced analytics only) |
| [docs/ui-framework-design.md](docs/ui-framework-design.md) | UI framework: shadcn/ui components, CSS variable theming, community themes |
| [docs/ui-pages-spec.md](docs/ui-pages-spec.md) | Page-by-page UI spec: every view, component, state, interaction |
| [docs/event-persistence-design.md](docs/event-persistence-design.md) | Event persistence, workflow checkpoints, SSE replay, Temporal migration path |
| [docs/DEVOPS.md](docs/DEVOPS.md) | DevOps: deploy, monitor, debug, CI/CD, AWS ops, secrets, troubleshooting |
| [docs/engine-gaps-analysis.md](docs/engine-gaps-analysis.md) | Engine gap analysis: 15 gaps prioritized P0-P2, fix sequencing |
| [docs/cc-supervision-design.md](docs/cc-supervision-design.md) | CC supervision: SupervisionConfig, review modes (BUILT) |
| [docs/map-collect-design.md](docs/map-collect-design.md) | MAP/COLLECT dynamic fan-out, TRANSFORM upgrade (BUILT) |
| [docs/media-store-architecture.md](docs/media-store-architecture.md) | S3 media store for universal HTTPS URLs (BUILT) |
| [README.md](README.md) | Quick start, feature overview, provider table |

## Reference: v1 Codebase

The original AgentOS is at `/Users/joelgreen/Development/Aiprojects/adalytics/`.
Reference for patterns and context. Do not copy code directly.
Key v1 files:
- `server/prisma/schema.prisma` — data model (agents, runs, tasks, credentials)
- `server/src/agent-runtime/` — executor patterns (local, queue, container)
- `server/src/routes/callbacks/` — Fargate callback pattern
- `client/src/components/ConductorChat.jsx` — Conductor chat UX
- `CLAUDE.md` — v1 project guide

## Dev Server

**Always use the dev-server script.** It auto-finds free ports so multiple CC instances don't collide.

```bash
./scripts/dev-server.sh          # start both servers on free ports
./scripts/dev-server.sh stop     # stop servers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joelgreen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
