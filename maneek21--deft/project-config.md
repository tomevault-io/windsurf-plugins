---
trigger: always_on
description: Deft is an open-source AI-native workspace. Native chat + tasks + an AI agent that plans and executes multi-step workflows across native data and connected calendar feeds and BYOA-provided external tools. The agent has direct SQL access to native data — not API calls — making it fundamentally faster and smarter than bolt-on AI features.
---

# AGENTS.md — Deft

## What is this?

Deft is an open-source AI-native workspace. Native chat + tasks + an AI agent that plans and executes multi-step workflows across native data and connected calendar feeds and BYOA-provided external tools. The agent has direct SQL access to native data — not API calls — making it fundamentally faster and smarter than bolt-on AI features.

One Next.js app. One Postgres database. Multi-tenant SaaS with org_id on every table.

Licensed under the GNU Affero General Public License v3.0 only (`AGPL-3.0-only`). Network deployments of modified versions must offer their users the corresponding source under AGPL section 13.

## Architecture

```
deft/
├── apps/
│   ├── web/          # Next.js 14 (App Router, TypeScript, Tailwind CSS)
│   └── api/          # Hono (TypeScript, REST endpoints, WebSocket via Socket.io)
├── packages/
│   ├── db/           # Drizzle ORM schema + client + migrations
│   └── shared/       # Shared types, Zod schemas, constants
├── docker-compose.yml  # Self-host: postgres + app
├── .env.example
├── LICENSE             # GNU AGPL v3.0 only
└── pnpm-workspace.yaml
```

**Stack:**
- Frontend: Next.js 14, App Router, TypeScript, Tailwind CSS, TipTap (editor)
- API: Hono on Node.js, TypeScript
- Database: PostgreSQL + pgvector (Drizzle ORM)
- Real-time: Socket.io in-process (single app instance; no cross-instance adapter)
- Auth: better-auth (JWT + refresh tokens). Google OAuth is retired from the self-hosted v1 product contract.
- Background jobs: PostgreSQL `job_queue` with in-process workers
- File storage: Cloudflare R2 or local (presigned uploads)
- AI: provider-neutral LLM routing. Anthropic, OpenAI/OpenAI-compatible, OpenRouter, and local Ollama-style providers are optional; core workspace flows must run without any provider key.
- Email: Resend (transactional)
- Monorepo: pnpm workspaces

## Database Design Principles

- `org_id` on EVERY table (multi-tenant, row-level isolation)
- Soft deletes everywhere (agent needs historical context)
- `created_at`, `updated_at` on every table
- UUIDs for primary keys (cuid2)
- All user-generated text stored as-is, never truncated
- Events table for connected tool data (unified schema)

## Code Conventions

- TypeScript strict mode everywhere
- Zod for all request/response validation
- Drizzle ORM — no raw SQL except in agent queries (agent needs direct access)
- API routes: `POST /api/spaces`, `GET /api/spaces/:id/messages`, etc.
- WebSocket events: `message:new`, `message:edited`, `typing:start`, `task:updated`
- Error responses: `{ error: string, code: string }` — never raw stack traces
- Components: functional React, no class components, prefer server components where possible
- Styling: Tailwind only, no CSS modules, no styled-components
- State: React hooks + context for client state, SWR or React Query for server state
- File naming: kebab-case for files, PascalCase for components

## Agent Architecture

The agent is NOT a chatbot. It's a workflow engine.

Agent engine lives in `apps/api/src/lib/` (agent-context, agent-plans, agent-tools, agent-actions, agent-runner, agent-stream-loop, agent-approval, agent-approval-resolver). The `packages/ai` stub was removed 2026-04-16.

**MCP Access is the pilot-facing integration surface.** Human employees create personal MCP tokens in Settings -> MCP Access and can connect Claude Desktop, Claude Code, ChatGPT MCP clients, or any streamable HTTP MCP client to `/api/mcp/v1`. Personal tokens act as the user who created them. Agent employees use the same MCP endpoint with employee tokens from Settings -> Agent Employees; those calls act as the employee and remain governed by trust, approval, health, and audit rules.

**Skills primitive (internal agent registry).** The `skills` table remains for first-party/internal agent-tool bundles and future ecosystem work, but it is not the pilot-facing onboarding primitive. Task templates are a separate first-class primitive (`task_templates` table) instantiated into any project via `POST /api/projects/:id/apply-template`. Project-level customization via `project_skills` / `skills.project_config` was retired 2026-04-18 in favor of fixed engineering defaults. See `docs/superpowers/specs/2026-04-18-simplify-skills-templates-design.md`.

**Observation pipeline:** Every chat message classified (Haiku): actionable? Intent? Entities? Urgency?

**Planner:** Complex requests decomposed into ordered steps. Plan shown to user → user edits/approves → agent executes with live progress (streamed to task-detail panel per Task 3.10) → pauses on failure or rolls back per plan mode.

**Proactive comments:** The nudge-check worker drops agent-authored comments on stalled/overdue tasks and on auto-accepted task extractions (Task 3.11), deduped 7d per task. Inline agent task-suggestion cards appear in chat for classified actionable messages (Task 3.12).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Maneek21/Deft](https://github.com/Maneek21/Deft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
