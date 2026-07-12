---
trigger: always_on
description: MCP-first customer platform made for the agentic era (KB, Conversations, CRM, CMS, Outreach). The agent is the UI: every action runs through MCP tools served at `/mcp`. There is no admin REST API for app data — the dashboard at `apps/web` is a thin shell that drives the same MCP endpoint.
---

# Munin — agent guide

MCP-first customer platform made for the agentic era (KB, Conversations, CRM, CMS, Outreach). The agent is the UI: every action runs through MCP tools served at `/mcp`. There is no admin REST API for app data — the dashboard at `apps/web` is a thin shell that drives the same MCP endpoint.

## Repository layout

Monorepo on pnpm + Turborepo.

- `apps/backend` — NestJS entry that composes `@getmunin/backend-core` with single-tenant `AuthModule`. Port 3001. Exposes `/mcp`, `/auth/*`, `/.well-known/oauth-*`, and the `/v1/*` control plane.
- `apps/web` — Next.js 15 dashboard + landing. Port 3000. Calls `apps/backend` via `/v1/*`.
- `apps/chat-widget` — embeddable browser bundle consumed via `mn_widget_*` keys.
- `packages/backend-core` — shared NestJS modules. Where business logic lives.
- `packages/dashboard-pages` — React pages reused by OSS and cloud webs.
- `packages/{core,db,types,sdk,mcp-toolkit,ui}` — non-Nest building blocks.
- `packages/{agent-host,agent-runtime}` — durable per-org LLM runner that drains the curator queue.
- `packages/widget-voice` — Vapi voice glue.

## Module map (`packages/backend-core/src/modules/`)

| Module | Tools prefix | Notes |
|---|---|---|
| `kb` | `kb_*` | Documents, spaces, hybrid search (BM25 + pgvector), curation candidates. |
| `conv` | `conv_*` | Email / SMS / voice / widget channels via the channel-adapter contract (`packages/backend-core/src/modules/conv/CLAUDE.md`). |
| `crm` | `crm_*` | Contacts, companies, deals, pipelines, segments, merge proposals. |
| `cms` | `cms_*` | Collections, entries, locales, assets, scheduled publishing, public delivery API. |
| `outreach` | `outreach_*` | Propose-only outbound campaigns. Never auto-sends. |
| `curator` | — | Background job queue (`curator_jobs`) running `skill://*` and `task://*` URIs. |
| `web` | — | Website scraper (single `task://web/scrape-website`). |
| `playbooks` | — | Cross-module packaged workflows (skill markdown only, no tools). |

Each module typically has `<mod>.module.ts`, `<mod>.service.ts`, `<mod>.tools.ts`, and a `skills/` directory of markdown procedures.

## MCP surface

- `@McpTool({ name, audiences, scopes, … })` decorator on Nest provider methods registers the tool. `audiences` gates by caller (admin vs end-user), `scopes` gates by OAuth scope (`kb:read`, `crm:write`, etc.).
- Skill markdown under `<module>/skills/<slug>.md` is auto-loaded by `skill-loader.ts` and surfaced as MCP `resources/list` URIs (`skill://<module>/<slug>`).
- Tool + skill enforcement lives in `packages/mcp-toolkit/src/server.ts`. Scopes are intersected against `actor.scopes` at call time.
- The same `/mcp` serves admin agents (OAuth-authorized) and end-user agents (delegated tokens minted via `delegated-token.controller.ts`).

### Adding a new MCP tool — checklist

Tools are a public product surface and are reviewed for the Anthropic Software Directory (reviewers call every tool with valid params and scan its metadata). Keep new tools correct *and* compliant:

- **Pre-check DB constraints — don't rely on `try/catch`.** A handler runs inside the request's outer tenant transaction, so a unique/FK violation poisons that transaction and the *commit* fails **after** your handler returns — past any in-handler catch — surfacing as a bare `500`. Guard *before* the failing statement: `SELECT` for an existing row (unique), check referencing rows before a delete (FK `restrict`), confirm an FK target exists before insert/update — then `throw new ConflictException('<module>_conflict: …')` / `BadRequestException(...)`. Pattern: `crm_create_pipeline` (unique), `crm_delete_segment` (FK), `conv_assign_conversation` (FK target). Never return a generic 500 — reviewers reject them.
- **Never return `void`/`undefined` from a handler.** `JSON.stringify(undefined)` is `undefined`, which fails the MCP `CallToolResult` schema (`-32602`). Return a small object, e.g. `{ deleted: true, id }`. (Dispatch coalesces void → `null` as a backstop, but return something meaningful.)
- **Annotations are required:** `title` plus exactly one hint — `readOnlyHint: true` for reads, `destructiveHint: true` for anything that writes/updates/deletes (read-only tools auto-run; destructive tools always prompt). Tool `name` ≤ 64 chars.
- **Split read and write.** No tool that both reads and mutates, and no catch-all `method`-style param; keep `create` / `update` / `delete` as separate tools.
- **Description = what it does and when to use it**, matching actual behavior. Don't tell Claude how to behave, don't direct it to call other tools, no hidden/encoded instructions — those are auto-rejected as prompt-injection.
- **Zod input schema** for every input (drives validation + the published JSON schema); set `audiences` (admin vs end-user) and `scopes` (`<module>:read` / `<module>:write`).
- **DB touch?** migration + RLS policy + per-module SQL (see Conventions).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getmunin/munin](https://github.com/getmunin/munin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
