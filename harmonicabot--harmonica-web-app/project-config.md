---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Harmonica is an LLM-powered deliberation and sensemaking platform. Users create "sessions" where groups can coordinate through AI-facilitated conversations, with automatic summarization and cross-pollination of ideas.

## Commands

```bash
npm run dev          # Start dev server at localhost:3000
npm run build        # Production build
npm start            # Run production build
npm run migrate      # Run database migrations
npm run migrate:down # Rollback migrations
```

```bash
# Evals (requires BRAINTRUST_API_KEY, ANTHROPIC_API_KEY, MAIN_LLM_* in .env.local)
npx tsx evals/facilitation.eval.ts       # Run synthetic facilitation evals
npx tsx evals/facilitation-real.eval.ts  # Run evals on real production sessions (last 14 days)
```

Note: No test or lint scripts are configured. TypeScript strict mode provides type safety. Node 20 required (`"engines": { "node": "20.x" }` in package.json). Use `npx tsc --noEmit` for type checking.

## Git Workflow

Always create a branch and open a PR for code changes — never commit directly to master. This applies to all application code changes; analytics-only or config changes outside the app don't require PRs.

```
production branch: master (NOT main — PRs to main don't deploy)
branch naming: feature/short-description, fix/short-description
```

## Architecture

### Tech Stack
- **Framework**: Next.js 14 (App Router)
- **Database**: Neon Postgres with Kysely query builder
- **Auth**: Auth0 (`@auth0/nextjs-auth0`, route at `src/app/api/auth/[auth0]/route.ts`)
- **LLM**: LlamaIndex with OpenAI/Anthropic/Google/PublicAI providers
- **Vector DB**: Qdrant for RAG queries
- **State**: Zustand (`src/stores/`)
- **UI**: Tailwind CSS + Radix UI + Shadcn components
- **Payments**: Stripe
- **Analytics**: PostHog
- **File Storage**: Vercel Blob

### Directory Structure

```
src/
├── app/                    # Next.js App Router pages
│   ├── (dashboard)/        # Authenticated dashboard routes
│   ├── api/                # API routes (see API Routes below)
│   ├── chat/               # Chat interface
│   ├── create/             # Session creation flow (4 steps)
│   ├── sessions/[id]/      # Session detail pages
│   └── workspace/[w_id]/   # Workspace pages
├── actions/                # Server actions (file uploads)
├── components/             # React components (Shadcn in ui/)
├── db/migrations/          # 33 Kysely migrations (000-031, gap at 002, collision at 025)
├── lib/
│   ├── monica/             # RAG/LLM query system
│   ├── schema.ts           # Database schema types
│   ├── db.ts               # Database queries
│   ├── modelConfig.ts      # LLM provider configuration
│   ├── permissions.ts      # Role-based access control
│   ├── crossPollination.ts # Cross-session idea sharing
│   └── defaultPrompts.ts   # Prompt templates
├── hooks/                  # React hooks
└── stores/                 # Zustand state stores
```

### API Routes

| Route | Purpose |
|-------|---------|
| `/api/auth/[...auth0]` | Auth0 authentication |
| `/api/builder` | Session prompt generation (CreatePrompt, EditPrompt, SummaryOfPrompt) |
| `/api/sessions` | Session CRUD |
| `/api/sessions/generate` | Generate session content |
| `/api/user/subscription` | Subscription management |
| `/api/llama` | LLM query endpoint |
| `/api/webhook/stripe` | Stripe webhooks (subscriptions, refunds → Discord notifications) |
| `/api/admin/prompts` | Admin prompt management (CRUD) |
| `/api/admin/prompt-types` | Prompt type management (CRUD) |
| `/api/admin/evals` | Braintrust experiment results (list + detail) |
| `/api/participant-suggestion` | Participant suggestions |
| `/api/sessions/[id]/generate-characters` | Generate conversation character personas |
| `/api/transcribe` | Audio transcription (Deepgram) |

### API v1 (`/api/v1/`)

Public REST API with Bearer token auth (`hm_live_` API keys). Routes in `src/app/api/v1/`:
- `src/lib/api/auth.ts` — `authenticateRequest()` supports both API key (Bearer) and Auth0 session
- `src/lib/api-types.ts` — Request/response types
- `src/lib/api/mappers.ts` — DB row → API response mappers
- `src/lib/api/errors.ts` — Standardized error responses

**Gotcha:** Under API key auth, `authGetSession()` returns null, so `insertHostSessions()` skips setting owner permission. Always call `setPermission(id, 'owner', 'SESSION', user.id)` explicitly after creating resources via the API.

### OpenAPI Spec

API spec is maintained in two places — both must be updated together:
- `docs/api-spec.yaml` (this repo)
- `harmonica-docs/api-reference/openapi.yaml` (Mintlify docs site)

### Core Concepts

**Sessions**: A "host session" is a deliberation created by an organizer. Each participant has a "user session" containing their conversation thread with the AI.

**Workspaces**: Container for organizing multiple sessions with custom visibility settings and banners.

**Monica**: The RAG system in `src/lib/monica/` handles intelligent querying across session data using Qdrant vector search. Supports single-session and multi-session queries.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/harmonicabot) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
