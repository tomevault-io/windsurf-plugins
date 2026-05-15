---
trigger: always_on
description: AI agents that passively watch business tools and produce reports via Claude. Open-source, serverless, runs on Cloudflare Workers.
---

# OpenChief

AI agents that passively watch business tools and produce reports via Claude. Open-source, serverless, runs on Cloudflare Workers.

## Architecture

- **pnpm monorepo + Turborepo** — `packages/shared`, `workers/*`, `workers/connectors/*`
- **Cloudflare Workers** — runtime (Durable Objects), router (Queue consumer), dashboard (React SPA + API), connectors (one Worker per data source)
- **Cloudflare D1** — shared database for agents, events, reports, revisions, identity mappings, model settings
- **Cloudflare KV** — caching layer for subscriptions, reports, user profiles, connector metadata, avatars
- **Cloudflare Queues** — `openchief-events` queue connects connectors to the router
- **Cloudflare Vectorize** — vector index for RAG (long-term memory), stores report/event embeddings
- **Cloudflare Workers AI** — embedding model (`@cf/baai/bge-base-en-v1.5`) for Vectorize indexing and retrieval
- **Durable Objects** — per-agent persistent state with SQLite inbox (events, reports, chat history, reasoning log)
- **React + Tailwind CSS v4** — SPA dashboard at `workers/dashboard`
- **Anthropic Claude** — LLM for report generation and agent chat (claude-sonnet-4-6 default)

## Repo Structure

```
openchief/
├── packages/shared/          # @openchief/shared — types, matching, ULID generation
├── workers/
│   ├── runtime/              # Agent Durable Object runtime (report generation + chat)
│   ├── router/               # Event router (queue consumer → D1 persistence)
│   ├── dashboard/            # React SPA + API worker (all UI + management endpoints)
│   ├── demo-engine/          # Synthetic event generator for demo mode (Serpin's Burger Shack)
│   └── connectors/           # One Worker per data source (14 connectors, all implemented)
│       ├── github/           # Webhook + polling
│       ├── slack/            # Webhook + polling + backfill + identity sync
│       ├── discord/          # Webhook (Ed25519) + polling
│       ├── figma/            # OAuth + webhook + polling (file updates, versions, comments, deletes, library publishes)
│       ├── jira/             # Polling (issues, transitions, sprints)
│       ├── jpd/              # Polling (Jira Product Discovery — ideas, insights)
│       ├── notion/           # Polling (pages, databases, comments)
│       ├── intercom/         # Webhook (HMAC-SHA1) + polling (conversations)
│       ├── twitter/          # OAuth PKCE + polling (multi-account, search)
│       ├── amplitude/        # Polling (metrics snapshots)
│       ├── google-calendar/  # OAuth + polling (calendar events)
│       ├── google-analytics/ # Polling (GA4 metrics via service account)
│       ├── quickbooks/       # OAuth (Intuit) + polling (invoices, payments, P&L)
│       └── rippling/         # Polling (employees, org structure, time-off)
├── agents/                   # 15 agent JSON definitions (data, not code)
├── migrations/               # 7 D1 SQL migration files
├── scripts/                  # setup.ts, seed-agents.ts, generate-config.ts, deploy.ts, teardown.ts
├── openchief.example.config.ts
├── turbo.json
└── pnpm-workspace.yaml
```

## Key Commands

```bash
pnpm build              # Build all packages (Turborepo)
pnpm typecheck           # Type check everything
pnpm dev                 # Start local dev servers
pnpm seed                # Seed agent definitions from agents/ to D1 (auto-detects --local vs --remote)
pnpm seed --remote       # Force seed to remote D1 (deployed instance)
pnpm seed --local        # Force seed to local D1 (wrangler dev)
pnpm run setup           # Interactive setup wizard (creates Cloudflare resources)
pnpm generate-config     # Generate wrangler.jsonc files from openchief.config.ts
pnpm run deploy          # Build + deploy all workers
pnpm run teardown        # Delete all workers + Cloudflare resources (--yes to skip prompts)

# Deploy individual workers
cd workers/runtime && npx wrangler deploy
cd workers/router && npx wrangler deploy
cd workers/dashboard && npx wrangler deploy
cd workers/connectors/github && npx wrangler deploy
cd workers/connectors/slack && npx wrangler deploy
```

## Event Flow

```
Source (GitHub, Slack, etc.)
  → Connector Worker (normalize to OpenChiefEvent, publish to queue)
    → Cloudflare Queue (openchief-events)
      → Event Router Worker (identity resolution, tweet enrichment, persist to D1)
        → Agent Durable Object reads events from D1 at report time
          → Claude generates report
            → Report stored in D1 + KV + local SQLite
```

## RAG (Long-Term Memory)

RAG gives agents historical context beyond the 48–72 hour event window. Implemented in `workers/runtime/src/rag.ts`.

### How It Works

1. After each report is generated, `indexReport()` embeds the headline and each section as separate vectors and upserts them to Cloudflare Vectorize (non-blocking via `ctx.waitUntil()`)
2. Before generating a report or answering a chat message, `retrieveContext()` embeds the query and searches Vectorize for the top-10 most relevant historical items, filtered by `agentId`
3. The retrieved context is injected into the prompt as `═══ HISTORICAL CONTEXT ═══`

### Cloudflare Services


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [serpin-taxt/openchief](https://github.com/serpin-taxt/openchief) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
