---
trigger: always_on
description: Matrix OS is **Web 4**: a unified AI operating system (OS + messaging + social + AI + games). Claude Agent SDK is the kernel. Everything persists as files. Reachable via web desktop, Telegram, WhatsApp, Discord, Slack, Matrix protocol. Vision: `specs/web4-vision.md`. Website: matrix-os.com
---

# AGENTS.md: Matrix OS

Matrix OS is **Web 4**: a unified AI operating system (OS + messaging + social + AI + games). Claude Agent SDK is the kernel. Everything persists as files. Reachable via web desktop, Telegram, WhatsApp, Discord, Slack, Matrix protocol. Vision: `specs/web4-vision.md`. Website: matrix-os.com

## Constitution

Read `.specify/memory/constitution.md`: the 9 core principles. Re-read after compaction.

Key principles:

1. **Data Belongs to Its Owner**: files hold identity/config/export state; user/org app data lives in owner-controlled Postgres
2. **AI Is the Kernel**: Agent SDK V1 `query()` with `resume`, model-agnostic routing over time
3. **Headless Core, Multi-Shell**: core works without UI, shell is one renderer
4. **Defense in Depth (NON-NEGOTIABLE)**: auth matrix, input validation, resource limits, timeouts
5. **TDD (NON-NEGOTIABLE)**: tests first, 99-100% coverage target

## Tech Stack

- **Runtime**: Node.js 24+, TypeScript 5.5+ strict, ES modules
- **AI**: Claude Agent SDK V1 `query()` + `resume`, Opus 4.6
- **Frontend**: Next.js 16 (`proxy.ts` replaces middleware, Turbopack, React Compiler, `cacheComponents`), React 19
- **Backend**: Hono (HTTP/WS gateway + channel adapters)
- **Database**: PostgreSQL via Kysely for platform, kernel durable state, social, app, and user data. Do not add SQLite/Drizzle/better-sqlite3 for new persistence.
- **Validation**: Zod 4 (`zod/v4` import)
- **Testing**: Vitest, `@vitest/coverage-v8`
- **Package Manager**: pnpm (install), bun (run scripts) -- NEVER npm

## SDK Decisions (Spike-Verified)

- V1 `query()` with `resume`: V2 silently drops mcpServers, agents, systemPrompt
- `createSdkMcpServer()` + `tool()` for in-process MCP tools (Zod 4 schemas)
- `allowedTools` is auto-approve, NOT filter: use `tools`/`disallowedTools` to restrict
- `bypassPermissions` propagates to ALL subagents: use PreToolUse hooks for access control
- Agent SDK bundles its own claude runtime -- no separate install needed
- `bypassPermissions` refuses root -- Docker must run as non-root user
- Prompt caching: `cache_control: {type: "ephemeral"}` on system prompt + tools (90% savings)
- Integration tests use haiku (<$0.10 per run)

## Development Rules

- **TDD**: failing tests FIRST, then implement (Red -> Green -> Refactor)
- **Conventional Commits and PR Titles**: commits and PR titles use semantic Conventional Commit style such as `feat(canvas): add workspace canvas`; never prefix PR titles with agent/tool tags like `[codex]`.
- **Specs go in `specs/`**: NEVER `docs/plans/`. Format: `specs/{NNN}-{feature-name}/`
- **Kysely/Postgres only**: never add SQLite, Drizzle ORM, or better-sqlite3 for new persistence
- **Kernel prompt**: keep under 7K tokens
- **Spike before spec**: test undocumented SDK behavior with throwaway code first
- After major features: run `/update-docs` to sync all documentation

## Mandatory Code Patterns

These patterns were identified as recurring defects across 4+ PRs (~317 unresolved review comments). Violations are the #1 source of bugs. Full analysis: `docs/dev/pr-review-analysis.md`

### Atomicity

- **2+ related DB writes MUST use a transaction**. No exceptions. Like + counter, delete + cascade, insert + update are all multi-step.
- **Optimistic concurrency must be enforced in the write statement**. Pre-reading a revision inside a transaction is not enough under READ COMMITTED; include `WHERE revision = :baseRevision` on the `UPDATE` or take a row lock.
- **Use `ON CONFLICT` for idempotent upserts** instead of check-then-insert (TOCTOU race).
- **Unique-scope create flows must be idempotent server-side**. If a unique index defines the logical singleton, `INSERT ... ON CONFLICT ... DO NOTHING` and select the existing row; do not rely on a client pre-check.
- **Use `{ flag: 'wx' }` for exclusive file creates** instead of `existsSync` + `writeFile`.

### External Calls

- **Every `fetch()` to an external service MUST have `signal: AbortSignal.timeout(ms)`**. Default: 10s for APIs, 30s for file downloads. No external call may hang indefinitely.
- **Server-side fetches of user-controlled URLs must block SSRF**. Parse the URL, resolve DNS, and reject loopback, link-local, private, multicast, documentation, and internal ranges before calling `fetch()`.
- **Server-side fetches of user-controlled URLs must reject redirects** unless each redirected URL is revalidated. Use `redirect: "error"` for preview/health checks to avoid redirect-based SSRF.
- **DNS preflight is not DNS pinning**. If user-controlled server-side fetch remains hostname-based after validation, document the residual DNS-rebinding risk or use a dispatcher/agent that pins the resolved address.
- **Never expose provider names or raw error messages to clients**. Log the real error server-side, return a generic message. This includes Postgres errors, Twilio/ElevenLabs/OpenAI errors, and filesystem paths.

### Input Validation

- **Use Hono `bodyLimit` middleware** on every mutating endpoint. Never check Content-Length after the body is already buffered.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HamedMP/matrix-os](https://github.com/HamedMP/matrix-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
