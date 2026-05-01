---
trigger: always_on
description: Chorus is an AI Agent & Human collaboration platform implementing the **AI-DLC (AI-Driven Development Lifecycle)** workflow. Multiple AI Agents (PM, Developer, Admin) and humans work together through a shared Idea → Proposal → Document + Task → Execute → Verify → Done pipeline.
---

# CLAUDE.md — Chorus Project Guide

## What is Chorus

Chorus is an AI Agent & Human collaboration platform implementing the **AI-DLC (AI-Driven Development Lifecycle)** workflow. Multiple AI Agents (PM, Developer, Admin) and humans work together through a shared Idea → Proposal → Document + Task → Execute → Verify → Done pipeline.

Core philosophy: **"Reversed Conversation"** — AI proposes, humans verify (not human prompt → AI execute).

## Tech Stack

- **Framework**: Next.js 15 (App Router, Turbopack for dev)
- **Language**: TypeScript 5 (strict mode)
- **Frontend**: React 19, Tailwind CSS 4, shadcn/ui (Radix UI)
- **Database**: PostgreSQL 16, Prisma ORM 7
- **Cache/Pub-Sub**: Redis 7 (ioredis, optional — falls back to in-memory)
- **Testing**: Vitest 4
- **Auth**: OIDC (users), API Keys with `cho_` prefix (agents), SuperAdmin (env-based bcrypt)
- **MCP**: @modelcontextprotocol/sdk 1.26 (HTTP Streamable Transport)
- **i18n**: next-intl (en, zh)
- **Package Manager**: pnpm 9.15
- **Path alias**: `@/*` → `./src/*`

## Project Structure

```
src/
├── app/                    # Next.js App Router
│   ├── (dashboard)/        # Main app layout (sidebar nav)
│   │   ├── projects/[uuid]/  # Project-scoped pages (tasks, ideas, proposals, docs)
│   │   └── settings/       # Agent API Key management, session management
│   ├── api/                # REST API routes + MCP endpoint
│   │   └── mcp/            # MCP HTTP streaming (POST init, DELETE close)
│   ├── admin/              # SuperAdmin panel
│   └── login/              # OIDC login flow
├── lib/                    # Core utilities (auth, prisma, api-response, uuid-resolver)
├── services/               # Business logic layer (all UUID-based)
├── mcp/                    # MCP Server factory + role-based tool modules
│   ├── server.ts           # Creates per-auth MCP server instance
│   └── tools/              # public.ts, developer.ts, pm.ts, admin.ts, session.ts
├── components/ui/          # shadcn/ui primitives
├── contexts/               # React contexts (locale)
├── i18n/                   # config.ts + request.ts
└── types/                  # TypeScript type definitions (auth.ts)

prisma/
├── schema.prisma           # 21 models, UUID-first architecture
└── migrations/             # DB migrations

messages/
├── en.json                 # English translations
└── zh.json                 # Chinese translations

public/skill/               # MCP Skill documentation served as static files
docs/                       # Architecture, PRD, MCP tools reference, design.pen
packages/chorus-cdk/        # AWS CDK for deployment
```

## Key Commands

```bash
pnpm dev                    # Dev server with Turbopack (:8637)
pnpm build                  # Production build (runs prisma generate first)
pnpm lint                   # ESLint
npx tsc --noEmit            # Type check
pnpm test                   # Run tests (Vitest)
pnpm test:watch             # Run tests in watch mode
pnpm db:migrate:dev         # Create/run dev migration
pnpm db:generate            # Regenerate Prisma client (REQUIRED after schema changes)
pnpm db:push                # Push schema to DB without migration (dev only)
pnpm db:studio              # Prisma Studio GUI (:5555)
pnpm docker:db              # Start PostgreSQL + Redis via Docker
docker compose up -d db     # Start PostgreSQL only (:5433)
```

## Architecture Patterns

### UUID-First

All entities use UUIDs as public identifiers. URLs, API params, and cross-entity references all use UUIDs. Never expose database serial IDs.

### Service Layer

Business logic lives in `src/services/*.service.ts`. API routes and MCP tools both call service functions — never put business logic directly in routes or tools.

### Auth Context

Every request resolves to an `AuthContext` with `type` ("user" | "agent" | "super_admin"), `companyUuid`, and `actorUuid`. The `getAuthContext(request)` function in `src/lib/auth.ts` checks: Bearer token (API Key or OIDC) → Session cookie (user_session / admin_session) → OIDC cookie (oidc_access_token).

Agent auth carries `roles: string[]` (pm_agent, developer_agent, admin_agent) which determines MCP tool visibility.

### Polymorphic Assignment

Tasks and Ideas use `assigneeType` ("user" | "agent") + `assigneeUuid` for flexible assignment to either humans or AI agents.

### MCP Server

The MCP endpoint at `POST /api/mcp` creates per-session server instances. Each session is tied to an authenticated agent. Tools are registered based on the agent's roles. Sessions auto-expire after 30 minutes of inactivity.

Tool registration pattern:
```typescript
server.registerTool("tool_name", {
  description: "...",
  inputSchema: z.object({ /* zod schema */ }),
}, async (params) => {
  const result = await someService.doSomething(auth.companyUuid, ...);
  return { content: [{ type: "text", text: JSON.stringify(result, null, 2) }] };
});
```

### Agent Sessions (Swarm Mode)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Chorus-AIDLC/Chorus](https://github.com/Chorus-AIDLC/Chorus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
