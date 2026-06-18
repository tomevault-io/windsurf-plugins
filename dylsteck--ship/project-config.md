---
trigger: always_on
description: Transforms SSE data into `UIMessage` — the single source of truth for all message state:
---

# AGENTS.md

This document provides context for AI agents working on the Ship codebase.

## Agent Skills

Project-specific skills live in `.agents/skills/`. **Reference and use these skills when they apply** — read the skill's `SKILL.md` file when a task matches its triggers.

| Skill | Purpose |
|-------|---------|
| **agent-browser** | Browser automation — navigate, fill forms, click, screenshot, scrape, test web apps |
| **ai-elements** | Create AI chat components in `packages/ui` following ai-elements patterns and shadcn/ui |
| **dogfood** | Systematic QA — explore apps, find bugs/UX issues, produce reports with repro evidence |
| **shadcn** | shadcn/ui components — add, search, fix, style, compose; use with `components.json` projects |

## Quick Start

```bash
pnpm install
pnpm dev
```

### Build

```bash
pnpm build        # Build all apps
pnpm typecheck   # Type check only
pnpm lint         # Lint all packages
```

> **After making changes:** always run `pnpm build && pnpm lint && pnpm typecheck` before finishing. Fix any errors before considering the task done.

### Deployment

The **API** runs on **Cloudflare Workers** (Wrangler). The **Next.js web app** (`apps/web`) is deployed as a **Docker** image (Next [standalone](https://nextjs.org/docs/app/api-reference/config/next-config-js/output) output), e.g. on [Coolify](https://coolify.io/docs/applications/nextjs).

#### Web App (Next.js) — Docker / Coolify

- **Dockerfile:** `apps/web/Dockerfile` (build context: **repository root**).
- **Port:** `3000` (set **Ports Exposes** to `3000` in Coolify).
- **Env:** Same variables as `apps/web/.env.example` (set in Coolify; pass build args for `NEXT_PUBLIC_*` / `API_BASE_URL` if needed at build time).

#### API (Cloudflare Worker) — Wrangler

Deploy from `apps/api`:

```bash
cd apps/api
npx wrangler deploy              # Deploy to production
npx wrangler deploy --env staging  # Deploy to staging (if configured)
npx wrangler dev                   # Local dev server
```

Secrets must be set via `wrangler secret put`:

```bash
npx wrangler secret put ANTHROPIC_API_KEY
npx wrangler secret put API_SECRET
npx wrangler secret put E2B_API_KEY
npx wrangler secret put OPENAI_API_KEY      # Optional, Codex API-key auth
npx wrangler secret put CODEX_AUTH_JSON      # Optional, personal ChatGPT sub (~/.codex/auth.json)
npx wrangler secret put CODEX_ACCESS_TOKEN  # Optional, enterprise Codex (see codex/auth docs)
```

## Ports

- Web App: `http://localhost:3000`
- API (local): `http://localhost:8787`

## Project Structure

```
ship/
├── apps/
│   ├── web/                          # Next.js App Router (frontend)
│   │   ├── app/(app)/dashboard       # Dashboard with chat UI
│   │   ├── components/chat/markdown.tsx  # Streamdown wrapper (animated fade-in)
│   │   ├── lib/
│   │   │   ├── session-logic.ts          # Pure timeline/approval/collapse derivations (Vitest)
│   │   │   ├── chat-store/               # Zustand streaming state + selectors
│   │   │   ├── session-connection/       # Unified WS + SSE lifecycle hooks
│   │   │   ├── ai-elements-adapter.ts    # SSE → UIMessage adapter
│   │   │   ├── sse-types.ts              # Wire-format event types (prefer @ship/contracts for new code)
│   │   │   └── api/                      # API client functions
│   │   └── components/               # Shared React components
│   └── api/                          # Cloudflare Worker (backend)
│       └── src/
│           ├── routes/
│           │   ├── chat.ts                       # Hono router for /chat/*
│           │   ├── chat-message-stream.ts        # POST /chat/:sessionId — drives one turn
│           │   ├── chat-auxiliary-routes.ts      # stop / subscribe / messages / git passthroughs
│           │   ├── chat-session-helpers.ts       # error persistence helpers
│           │   ├── sessions.ts                   # Session CRUD
│           │   ├── sandbox.ts                    # Sandbox management
│           │   ├── models.ts                     # Model listing (powered by agent-registry)
│           │   ├── git.ts                        # Git operations
│           │   ├── connectors.ts                 # GitHub connector status/enable/disable
│           │   ├── terminal.ts                   # Terminal access
│           │   └── openapi.ts                    # GET /openapi.json (public spec)
│           ├── openapi/
│           │   ├── schemas.ts                    # Zod REST schemas (OpenAPI + route validation)
│           │   └── build-spec.ts                 # Programmatic OpenAPI 3.1 document
│           ├── lib/
│           │   ├── acp-chat-runner.ts           # One turn — ACP JSON-RPC over bridge → Ship SSE
│           │   ├── acp-bridge-bootstrap.ts      # Bundled bridge drop-in + `/healthz` polling
│           │   ├── acp-json-rpc.ts              # WebSocket envelopes + JSON-RPC multiplexing
│           │   ├── chat-runner.ts               # Facade re-export (stable import path)
│           │   ├── chat-workspace.ts            # Sandbox + repo provisioning for a turn
│           │   ├── chat-history.ts              # Persisted rows → `ChatTurnMessage[]`
│           │   ├── chat-stream-helpers.ts       # `writeStatus` / `writeError` / `writeDone`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dylsteck/ship](https://github.com/dylsteck/ship) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
