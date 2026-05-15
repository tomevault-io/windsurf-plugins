---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Starting Development
```bash
pnpm dev                # Start both frontend and backend (recommended)
pnpm dev:frontend       # Start only Next.js dev server on port 3000
pnpm dev:backend        # Start only Convex dev server
```

The `predev` script automatically runs before `pnpm dev` to ensure Convex is ready and opens the dashboard.

### Building and Linting
```bash
pnpm build             # Build Next.js for production
pnpm lint              # Run ESLint on the codebase
```

### Reviewing Code
```bash
run coderabbit --prompt-only
let it run as long as it needs (run it in the background) and fix any issues.
```


### Convex Operations
```bash
npx convex dev         # Start Convex development server and watch for changes
npx convex deploy      # Deploy Convex backend to production
npx convex env set KEY value    # Set environment variable in Convex
npx convex env list    # List all Convex environment variables
```

## Architecture Overview

### Tech Stack
- **Frontend**: Next.js 14 (App Router), TypeScript, React 18, Tailwind CSS
- **UI Components**: Shadcn/ui (built on Radix UI primitives)
- **Backend**: Convex (serverless real-time database with built-in functions)
- **Authentication**: Convex Auth with OAuth (GitHub, Google)
- **AI Integration**: Heroku Managed Inference with multi-model support
- **Package Manager**: pnpm 10.x (required)

### Directory Structure
- `app/` - Next.js 14 App Router structure
  - `(protected)/dashboard/` - Protected routes requiring authentication
  - `api/` - API route handlers
- `convex/` - Convex backend functions, schema, and authentication
- `components/` - Reusable React components
- `lib/` - Utility functions and shared logic
- `hooks/` - Custom React hooks
- `store/` - Zustand state management
- `types/` - TypeScript type definitions

### Key Architectural Patterns

#### 1. Convex Backend Architecture
Convex provides a serverless real-time backend with:
- **Schema-first design**: Database schema defined in `convex/schema.ts` with strong typing
- **Queries and Mutations**: Functions in `convex/*.ts` files are automatically exposed as API endpoints
- **Real-time subscriptions**: Frontend automatically receives updates when data changes
- **Authentication**: Convex Auth handles OAuth and session management

Main backend files:
- `convex/schema.ts` - Database schema with tables and indexes
- `convex/study.ts` - Study session management and coin system
- `convex/groups.ts` - Study group management
- `convex/messages.ts` - Group chat messages
- `convex/todos.ts` - Task management
- `convex/auth.ts` - Authentication logic

#### 2. MentorMind AI System (RAG Architecture)
The AI assistant uses a sophisticated Retrieval-Augmented Generation (RAG) pipeline:

**Context Retrieval Flow:**
1. User query triggers `app/api/ai-helper/route.ts`
2. System fetches user context from Convex:
   - Study statistics (total time, session history)
   - Performance metrics (completion rate, success patterns)
   - Group memberships and collaboration data
3. Context is injected into system prompt via `_lib/system-prompt.ts`

**Model Routing:**
- Multi-model support (GPT-OSS 120B, Nova Lite/Pro, Claude 3.5 Haiku)
- Automatic model selection based on query complexity
- Manual model override available via UI
- Routing logic in `app/api/ai-helper/_lib/model-router.ts`

**MCP Tool Integration:**
- MCP tools are dynamically discovered from `/v1/mcp/servers` endpoint
- All available tools are provisioned to the AI agent
- AI decides when to use tools (e.g., `fetch/read_url` for external content)
- Heroku Agents API endpoint handles tool invocations
- SSE (Server-Sent Events) response parsing for streaming

**Key files:**
- `app/api/ai-helper/route.ts` - Main AI endpoint with MCP integration
- `app/api/ai-helper/_lib/system-prompt.ts` - RAG context builder
- `app/api/ai-helper/_lib/model-router.ts` - Model selection logic
- `app/api/ai-helper/_lib/openai-client.ts` - Heroku API client

#### 3. Coin Economy System
Users earn and spend coins as a gamification mechanism:
- **Earning**: 1 coin per second of completed study time (defined in `convex/study.ts:10`)
- **Spending**: 100 coins per AI query (defined in `convex/study.ts:11` as `AI_HELPER_QUERY_COST`)
- **Initial balance**: 500 coins for new users (defined in `convex/study.ts:9`)
- **Implementation**:
  - Coins earned in `convex/study.ts:completeSession`
  - Coins spent via `convex/study.ts:spendCoins`
  - Refunds handled via `convex/study.ts:refundCoins`

**Important**: When modifying coin costs, update both the backend constant and any frontend references.

#### 4. Authentication Flow
- OAuth providers configured via Convex environment variables
- Callback URLs follow pattern: `https://<deployment>.convex.site/api/auth/callback/<provider>`
- Setup script (`setup.mjs`) guides through OAuth configuration
- User sessions managed automatically by Convex Auth
- Protected routes use `getAuthUserId(ctx)` in Convex functions

#### 5. Spotify Integration
The app includes Spotify OAuth for music playback during study sessions:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prime399/study-flow](https://github.com/prime399/study-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
