---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Splitrail Cloud is a Next.js web application that serves as the cloud platform for [Splitrail](https://github.com/Piebald-AI/splitrail), an analyzer for agentic AI coding tool usage. Users upload anonymized usage statistics from the Splitrail CLI to track their development metrics, view personal analytics, and optionally participate in a public leaderboard.

**Live URL**: https://splitrail.dev

## Tech Stack

- **Framework**: Next.js 15.3.5 with App Router
- **Language**: TypeScript with strict mode
- **Database**: PostgreSQL with Prisma ORM
- **Authentication**: NextAuth.js 5.0 (beta) with GitHub OAuth
- **Styling**: Tailwind CSS v4 with Radix UI components
- **State Management**: TanStack Query (React Query)
- **Monitoring**: Sentry for error tracking
- **Analytics**: Vercel Analytics, PostHog
- **Package Manager**: **pnpm** (never use npm or yarn)

## Development Commands

```bash
# Development
pnpm dev              # Start dev server with Turbopack
pnpm build           # Build for production (runs prisma generate first)
pnpm start           # Start production server
pnpm lint            # Run ESLint
pnpm format          # Run Prettier
pnpm type-check      # TypeScript type checking (tsc --noEmit)

# Database
pnpm db:generate     # Generate Prisma client
pnpm db:push         # Push schema to database (use for dev)
pnpm db:migrate      # Run migrations (use for production)
pnpm db:migration:new # Create new migration
pnpm db:seed         # Seed database
pnpm db:studio       # Open Prisma Studio
pnpm db:reset        # Reset database (force)
```

## Path Aliases

TypeScript is configured with the `@/*` path alias mapping to `./src/*`. Always use this alias for imports:

```typescript
import { db } from "@/lib/db";
import { Stats } from "@/types";
```

## Critical Architecture Concepts

### Multi-Application Support

The platform tracks statistics from multiple agentic AI coding tools, not just Claude Code. Supported applications are defined in `src/types/index.ts` and `src/lib/application-config.ts`:

- `claude_code` - Claude Code
- `open_code` - OpenCode
- `cline` - Cline (VSCode extension)
- `roo_code`, `kilo_code`, `qwen_code` - Other AI coding assistants
- `codex_cli`, `gemini_cli`, `kilo_cli` - Alternative CLIs
- `copilot` - GitHub Copilot
- `pi_agent` - Pi Agent

**Important**: When adding features, always ensure they work across all applications, not just Claude Code.

### Stats Aggregation System

Statistics are aggregated across multiple time periods for efficient querying:

- **MessageStats** (table: `message_stats`) - Raw message-level data from CLI uploads. Each row represents one message in a conversation.
- **UserStats** (table: `user_stats`) - Aggregated statistics per user, per application, per period.
  - Periods: `hourly`, `daily`, `weekly`, `monthly`, `yearly`
  - Enables fast leaderboard queries without scanning all messages

When uploading stats via `/api/upload-stats`, the system:

1. Stores raw MessageStats
2. Aggregates into UserStats for all time periods
3. Uses upsert pattern to update existing aggregations

### Database Schema Key Points

All statistics fields use `BigInt` for large numbers except `cost` which is `Float`. The schema is split between:

- `prisma/schema.prisma` - Database schema definition
- `src/types/index.ts` - TypeScript types (must be kept in sync)

**Critical**: When modifying MessageStats or UserStats schema, update BOTH files and check the comment at the top of the model in schema.prisma.

### API Token Authentication

CLI access uses bearer token authentication:

- Tokens are prefixed with `st_` (splitrail token)
- Generated and managed in the Settings page (`/settings`)
- Stored in the `api_tokens` table
- Validated via `Authorization: Bearer <token>` header
- Last used timestamp tracked on each request

### Statistics Field Categories

Stats are categorized for UI display and filtering:

- **File Operations**: filesRead, filesAdded, filesEdited, filesDeleted
- **Line Operations**: linesRead, linesAdded, linesEdited, linesDeleted
- **Token Usage**: inputTokens, outputTokens, cacheCreationTokens, cacheReadTokens, cachedTokens, reasoningTokens
- **Tool Usage**: toolCalls, terminalCommands, fileSearches, fileContentSearches
- **Content Types**: codeLines, docsLines, dataLines, mediaLines, configLines, otherLines
- **Todo Tracking**: todosCreated, todosCompleted, todosInProgress, todoReads, todoWrites
- **Bytes**: bytesRead, bytesAdded, bytesEdited, bytesDeleted
- **Cost**: cost (in USD, converted to user's preferred currency in UI)

Reference `src/types/index.ts` for the canonical list of stat keys.

## Key API Endpoints

### POST /api/upload-stats

**Primary CLI integration endpoint**. Accepts an array of ConversationMessage objects containing statistics data.

- Authentication: Bearer token in Authorization header
- Request body: Array of ConversationMessage (see `src/types/index.ts`)
- Processes messages in batches to aggregate into UserStats
- Updates `lastUsed` timestamp on API token

### Token Management

- `GET /api/user/token` - List user's API tokens

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Piebald-AI/splitrail-cloud](https://github.com/Piebald-AI/splitrail-cloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
