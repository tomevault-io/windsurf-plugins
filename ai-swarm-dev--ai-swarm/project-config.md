---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI Swarm is an autonomous development orchestration system powered by LLMs and Temporal.io. Unlike simple AI coding assistants, AI Swarm manages the entire software development lifecycle—from planning and coding to review and deployment—with durable execution, automatic retries, and Git branch isolation.

## Development Commands

```bash
# Root level (turbo monorepo)
pnpm dev          # Start all services in development mode
pnpm build        # Build all packages
pnpm test         # Run tests across all packages
pnpm lint         # Lint all code
pnpm format       # Format with Prettier

# Individual services
pnpm worker       # Start worker service (packages/worker)
pnpm portal       # Start portal dashboard (apps/portal)
pnpm cli          # Start CLI tool (apps/cli)

# Docker-based development (recommended for local testing)
docker compose up                       # Start all services
docker compose -f docker-compose.yml -f docker-compose.local.yml up -d  # Development mode
docker logs -f ai-swarm-worker-1        # View worker logs
docker logs -f ai-swarm-portal          # View portal logs
```

## High-Level Architecture

AI Swarm follows a **decoupled, event-driven architecture** designed for reliability and scalability:

### Core Components

1. **Temporal.io Orchestration** (`packages/workflows/`)
   - The "brain" of the system—manages state persistence, timeouts, and retries
   - Workflows durable across server restarts
   - Main workflow: `developFeature` in `packages/workflows/src/workflows/develop-feature.ts`

2. **Stateless Workers** (`packages/worker/`)
   - Execute Temporal activities (planning, coding, testing, deployment)
   - Ephemeral and horizontally scalable (1-8 workers via `./scale-workers.sh`)
   - Entry point: `packages/worker/src/index.ts`

3. **Portal UI** (`apps/portal/`)
   - Next.js 14 dashboard for task submission and workflow monitoring
   - Real-time status updates via Redis pub/sub
   - Passkey (WebAuthn) authentication—no external IdP dependencies

4. **Shared Package** (`packages/shared/`)
   - Type definitions, LLM client, database client, services
   - SCM integration (GitHub, GitLab, Azure DevOps)
   - Context discovery for `.aicontext/` and `claude.md` files

### Services Architecture (Docker)

```
┌─────────────────────────────────────────────────────────────────┐
│                        AI Swarm Stack                            │
├─────────────────────────────────────────────────────────────────┤
│  temporal          ──  Workflow orchestration engine             │
│  temporal-ui       ──  Workflow visualization UI                  │
│  postgres          ──  Temporal persistence                      │
│  redis             ──  Caching and worker health monitoring       │
│  worker (scaled)   ──  Temporal worker processes (1-8 replicas)  │
│  portal            ──  Next.js dashboard UI                      │
│  builder           ──  Persistent build environment               │
│  playwright        ──  Browser testing service                   │
│  socket-proxy      ──  Secure Docker socket access               │
└─────────────────────────────────────────────────────────────────┘
```

## Workflow Lifecycle

Every task follows a rigorous multi-stage pipeline (defined in `packages/workflows/src/workflows/develop-feature.ts`):

1. **Planning** (`planTask`) — Analyzes codebase context and decomposes task into actionable steps
2. **Worktree Creation** (`createWorktree`) — Isolated Git worktree for safe development
3. **Coding** (`executeCode`) — Implements changes with atomic self-correction (up to 3 attempts)
4. **Shadow Review** (`reviewCode`) — Autonomous code review against plan and standards
5. **Build Verification** (`verifyBuild`) — Runs build and tests in isolated environment
6. **PR Merge** (`mergePullRequest`) — Squash merge to main with branch deletion
7. **Deployment** (`deployToProduction`) — LLM-powered intelligent deployment with rollback on failure
8. **Post-Deploy Verification** (`verifyDeployment`) — Live verification of production deployment

## LLM Integration

AI Swarm uses a **multi-role LLM strategy** with model cascades:

| Role | Default Provider | Models | Purpose |
|------|------------------|--------|---------|
| `planner` | Gemini | `gemini-2.5-pro`, `gemini-2.5-flash` | Deep reasoning for task planning |
| `coder` | Claude/Gemini | Configurable | Code implementation |
| `reviewer` | Gemini | `gemini-2.5-pro` | Code review and quality checks |
| `deployer` | Gemini | `gemini-2.5-flash`, `gemini-2.5-pro` | Deployment troubleshooting |
| `portal_planner` | Gemini | `gemini-2.5-flash` | Fast chat responses in Portal |

**Configuration:** Set `LLM_{ROLE}=claude|gemini` environment variable per role. Defaults to Gemini for most roles.

**Claude Integration:** Uses Z.ai API proxy (`Z_AI_API_KEY`). OAuth credentials shared via `workers_oauth` volume.

## Key Patterns

### 1. Temporal Activities

Activities in `packages/workflows/src/activities/` are **individual units of work** executed by workers:

- `planner.ts` — `planTask`, `reviewCode`
- `coder.ts` — `executeCode`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ai-swarm-dev/ai-swarm](https://github.com/ai-swarm-dev/ai-swarm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
