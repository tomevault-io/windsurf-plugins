---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Key Commands

### Development

- `bun dev` - Start development server with Turbopack (http://localhost:3000)
- `bun run build` - Create production build
- `bun start` - Start production server
- `bun lint` - Run ESLint for code quality checks and import sorting
- `bun typecheck` - Run TypeScript type checking
- `bun format` - Format code with Prettier
- `bun format:check` - Check code formatting without modifying files
- `bun migrate` - Run database migrations
- `bun run test` - Run tests with Vitest (use this, NOT `bun test`)
- `bun run test:watch` - Run tests in watch mode

### Docker

- `docker compose up -d` - Run the application with Docker (includes PostgreSQL 17, Redis, Morphic app, and SearXNG)
- `docker compose down` - Stop all containers
- `docker compose down -v` - Stop all containers and remove volumes (deletes database data)
- `docker pull ghcr.io/miurla/morphic:latest` - Pull prebuilt Docker image

#### Docker Authentication

**Default Behavior**: Docker deployments run in **anonymous mode** (authentication disabled).

When running with Docker Compose, `ENABLE_AUTH=false` is set by default, allowing personal use without Supabase setup. All users share a single anonymous user ID.

**⚠️ Security Warning:**

- Anonymous mode is **only for personal, single-user local environments**
- All chat history is shared under one user ID
- **NOT suitable** for multi-user or production deployments
- Morphic Cloud deployments block `ENABLE_AUTH=false` automatically

**Enabling Authentication:**
To require Supabase authentication, set:

```bash
ENABLE_AUTH=true  # or remove ENABLE_AUTH from docker-compose.yaml
NEXT_PUBLIC_SUPABASE_URL=[your-supabase-url]
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=[your-supabase-publishable-key]
SUPABASE_SECRET_KEY=[your-supabase-secret-key]
```

**Implementation:**

- Auth logic: [lib/auth/get-current-user.ts:22-40](lib/auth/get-current-user.ts#L22-L40)
- Always warns when `ENABLE_AUTH=false` (except in tests)
- Guards against `MORPHIC_CLOUD_DEPLOYMENT=true`

## Architecture Overview

### Tech Stack

- **Next.js 16.2.1** with App Router, React Server Components, and Turbopack
- **React 19.2.0** with TypeScript for type safety
- **Vercel AI SDK 5.0.0-alpha.2** for AI streaming and GenerativeUI
- **Supabase** for authentication and backend services
- **PostgreSQL** with Drizzle ORM for database and chat history storage
- **Redis** (Upstash or local) for SearXNG advanced search caching
- **Tailwind CSS** with shadcn/ui components

### Core Architecture

1. **App Router Structure** (`/app`)
   - `/api/` - Backend API routes for chat, search, and auth endpoints
   - `/auth/` - Authentication pages (login, signup, password reset)
   - `/search/` - Search functionality and results display
   - `/share/` - Sharing functionality for search results

2. **AI Integration** (`/lib`)
   - `/lib/agents/` - AI agents for research and question generation
   - `/lib/config/` - Model configuration management
   - `/lib/streaming/` - Stream handling for AI responses
   - `/lib/tools/` - Search and retrieval tool implementations
   - Models configured in `public/config/models.json`

3. **Database** (`/lib/db`)
   - PostgreSQL database with Drizzle ORM
   - Schema defined in `/lib/db/schema.ts`
   - Migrations in `/lib/db/migrations/`
   - Database actions in `/lib/actions/chat-db.ts`

4. **Search System**
   - Multiple providers: Tavily (default), SearXNG (self-hosted), Exa (neural), Brave (optional)
   - Brave Search is optional; if API key is not provided, type="general" searches fall back to primary provider
   - Video/image search support depends on configured providers (Brave provides best multimedia support)
   - URL-specific search capabilities
   - Configurable search depth and result limits

5. **Component Organization** (`/components`)
   - `/artifact/` - Search result and AI response display components
   - `/sidebar/` - Chat history and navigation
   - `/ui/` - Reusable UI components from shadcn/ui
   - Feature-specific components (auth forms, chat interfaces)

6. **State Management**
   - Server-side state via React Server Components
   - Client-side hooks in `/hooks/`
   - Redis for persistent chat history
   - Supabase for user data

## Environment Configuration

### Required Variables

```bash
OPENAI_API_KEY=      # Default AI provider
TAVILY_API_KEY=      # Default search provider
DATABASE_URL=        # PostgreSQL connection string
```

### Optional Features

- Chat history: Set `ENABLE_SAVE_CHAT_HISTORY=true` and configure Redis
- Alternative AI providers: Add corresponding API keys (ANTHROPIC_API_KEY, GOOGLE_GENERATIVE_AI_API_KEY, etc.)
- Alternative search: Configure SEARCH_API and provider-specific settings
- Sharing: Set `NEXT_PUBLIC_ENABLE_SHARE=true`

## Key Development Patterns

1. **AI Streaming**: Uses Vercel AI SDK's streaming capabilities for real-time responses
2. **GenerativeUI**: Dynamic UI components generated based on AI responses
3. **Type Safety**: Strict TypeScript configuration with comprehensive type definitions in `/lib/types/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [miurla/morphic](https://github.com/miurla/morphic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
