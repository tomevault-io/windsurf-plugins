---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**HotHotLeads** is an AI-led GTM (Go-To-Market) Engineering engine built on NestJS + Claude. It:
- Reads and writes product knowledge (ICP, strategy, channels, pricing) in a **Postgres** database managed via MikroORM in [packages/db](packages/db/)
- Discovers, enriches, scores leads and drafts outreach using **Claude multi-agent pipelines**
- Monitors GTM signals via **Exa** web research
- Is controlled via **Telegram** and **WhatsApp** — both on-demand and on schedule
- Persists all output (leads, outreach, signals, feedback) in Postgres

## Commands

All commands run from the repo root and are orchestrated by Turborepo.

```bash
# Install all workspace dependencies (one node_modules at the root)
npm install

# One-time: start Postgres and apply migrations
docker compose up -d postgres
npm --workspace @hothotleads/db run db:migrate

# Development — starts both apps in watch mode
npx turbo run dev

# Run only the API in watch mode
npx turbo run dev --filter=@hothotleads/api

# Run only the web app
npx turbo run dev --filter=@hothotleads/web

# Production build (packages, then apps, in dependency order)
npx turbo run build

# Type-check the entire workspace
npx turbo run typecheck

# Lint the entire workspace
npx turbo run lint

# Run all tests
npx turbo run test

# Run a single API test file
npm --workspace @hothotleads/api run test:single -- src/agents/orchestrator/orchestrator.agent.spec.ts

# Start the API from its build output
npm --workspace @hothotleads/api run start
```

## Architecture

This is a Turborepo monorepo with apps under `apps/` and shared packages under `packages/`.

```
apps/
├── api/                            # NestJS API + agent orchestration (@hothotleads/api)
│   ├── package.json
│   ├── tsconfig.json               # extends @hothotleads/config/tsconfig/nest.json
│   ├── nest-cli.json
│   ├── .env                        # runtime secrets (gitignored)
│   └── src/
│       ├── main.ts                 # NestJS bootstrap
│       ├── app.module.ts           # Root module wiring
│       ├── health.controller.ts    # GET /health
│       ├── config/configuration.ts # All env vars typed
│       ├── common/types/           # Shared TypeScript interfaces
│       ├── agents/                 # BaseAgent + 8 specialized agents
│       ├── channels/               # Telegram + WhatsApp webhook controllers
│       ├── integrations/           # Exa / Tavily / Perplexity web-research clients
│       ├── db/db.module.ts         # MikroORM wiring + repository providers
│       └── scheduler/              # Cron-registration service (loads schedules from Postgres)
└── web/                            # Vite + React operator UI (@hothotleads/web)
    ├── package.json
    ├── tsconfig.json               # extends @hothotleads/config/tsconfig/vite.json
    ├── vite.config.ts
    └── src/                        # pages, components, hooks

packages/
├── config/                         # Shared TSConfig / ESLint / Prettier bases (@hothotleads/config)
│   ├── tsconfig/{base,nest,vite,library}.json
│   ├── eslint/base.js
│   └── prettier/index.js
├── types/                          # Shared domain types (@hothotleads/types) — plain interfaces shared between api + web
└── db/                             # Postgres schema + entities + repositories (@hothotleads/db)

turbo.json                          # pipeline: build, dev, lint, typecheck, test, start
```

**Dependency rule:** apps may depend on packages. Packages must not depend on apps. Apps must not import from sibling apps.

## Agent Architecture

8 specialized agents, all extending `BaseAgent` (which owns the Claude tool-use loop):

| Agent | Role | Key Tools |
|-------|------|-----------|
| **OrchestratorAgent** | Entry point — parses intent, loads product context, routes to specialists | get_products, load_product_context, run_* |
| **ProductContextAgent** | Loads + summarizes a product's knowledge (ICP, channels, sources, predefined prospects) from Postgres | get_product_knowledge |
| **ProspectDiscoveryAgent** | Finds new companies matching ICP via Exa | search_companies, save_lead |
| **EnrichmentAgent** | Deep-researches a company (news, funding, hiring, tech stack) | research_company, search_web, update_lead |
| **ScoringAgent** | Scores leads 0–100 against ICP + signals | get_product_knowledge, update_lead_score |
| **OutreachAgent** | Drafts channel-specific personalized messages referencing signals | get_lead, save_outreach |
| **SignalMonitorAgent** | Monitors Sources for new GTM signals and writes them to Notion | search_signals, save_signal |
| **FeedbackAgent** | Evaluates output quality (0–10) and writes to Feedback Log | get_recent_feedback, save_feedback |

### Feedback Loop
Every significant output (lead batch, outreach draft) automatically triggers **FeedbackAgent** via `OrchestratorAgent.executeTool('run_feedback', ...)`. Feedback is persisted in the `feedback_entries` table (scoped by `product_id`). Run `/feedback-review` to surface patterns and ICP refinement suggestions.

## Data Model


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [harsimarriar96/hot-hot-leads](https://github.com/harsimarriar96/hot-hot-leads) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
