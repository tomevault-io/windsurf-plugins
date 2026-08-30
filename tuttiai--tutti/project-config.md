---
trigger: always_on
description: This file is read by Claude Code on every session. These are rules, not suggestions.
---

# CLAUDE.md — Law of the Tutti Codebase

This file is read by Claude Code on every session. These are rules, not suggestions.

---

## Pre-flight Checklist

Before every edit, verify ALL of the following:

- [ ] No `any` type introduced — use `unknown` + type guards
- [ ] No `process.env` — use `SecretsManager.require()` / `.optional()`
- [ ] No API keys in logs, events, errors, or tool results
- [ ] All tool results wrapped with `PromptGuard.wrap()`
- [ ] Dependency direction respected: types <- core <- cli, types <- voices
- [ ] Every new public method has at least one unit test
- [ ] Conventional Commit message with package scope
- [ ] `npm audit --audit-level=high` passes
- [ ] CHANGELOG.md updated under `[Unreleased]`
- [ ] Voice `execute()` never throws — returns `{ content, is_error: true }`
- [ ] TSDoc on every new export
- [ ] No `console.log` — use the pino logger

---

## 1. Project Overview

Tutti is an open-source multi-agent orchestration framework for TypeScript.

### Monorepo structure

```
packages/types/      @tuttiai/types      Interfaces and Zod schemas (ZERO runtime deps)
packages/core/       @tuttiai/core       Runtime, agent loop, providers, security
packages/cli/        @tuttiai/cli        Binary: tutti-ai
packages/server/     @tuttiai/server     HTTP server: REST API + SSE streaming
packages/router/     @tuttiai/router     Smart model router (heuristic + LLM classifier)
packages/telemetry/  @tuttiai/telemetry  OpenTelemetry tracer, cost estimation, exporters
packages/inbox/      @tuttiai/inbox      Inbound messaging orchestrator — dispatches platform messages to agents
packages/tutti-ai/   tutti-ai            Thin wrapper re-exporting the CLI binary
voices/filesystem/   @tuttiai/filesystem 7 file system tools
voices/github/       @tuttiai/github     10 GitHub API tools
voices/playwright/   @tuttiai/playwright 12 browser automation tools
voices/mcp/          @tuttiai/mcp        MCP bridge — wraps any MCP server
voices/web/          @tuttiai/web        3 web tools: search, fetch, sitemap
voices/sandbox/      @tuttiai/sandbox    4 tools: execute, read, write, install
voices/slack/        @tuttiai/slack      11 Slack workspace tools (chat, reactions, channels, users)
voices/discord/      @tuttiai/discord    11 Discord tools (messages, channels, members, reactions, DMs)
voices/postgres/     @tuttiai/postgres   8 Postgres tools (query/execute + introspection)
voices/stripe/       @tuttiai/stripe     27 Stripe API tools (customers, payments, subs, invoices, balance)
voices/twitter/      @tuttiai/twitter    9 Twitter / X tools (tweets, threads, mentions, timeline)
voices/telegram/     @tuttiai/telegram   Telegram tools (send, edit, delete, react) + shared bot client for @tuttiai/inbox
voices/email/        @tuttiai/email      Email tools (send, reply, list_inbox) + IMAP IDLE / SMTP shared client for @tuttiai/inbox
voices/whatsapp/     @tuttiai/whatsapp   WhatsApp Cloud API tools (send_text, send_template) + signed-webhook listener for @tuttiai/inbox
voices/rag/          @tuttiai/rag        4 RAG tools (ingest, search, list sources, delete source)
docs/                                    Astro Starlight documentation site
```

### Key invariants — NEVER violate

- `packages/types` has **zero** runtime dependencies (only `zod`).
- Voices **never** import from `packages/core` (except `@tuttiai/core` for logging utilities).
- **No** circular dependencies between packages.
- Every exported symbol has a TSDoc comment.

### Terminology

| Term | Definition |
|------|-----------|
| **Voice** | Pluggable module giving an agent tools. Implements the `Voice` interface. |
| **Score** | Top-level config file (`tutti.score.ts`). Defines agents, provider, model, memory, telemetry. |
| **Agent** | Named LLM persona with system prompt, model, and voices. |
| **Tool** | Single callable function. Zod schema + `execute()` handler. |
| **Repertoire** | Voice registry at `github.com/tuttiai/voices`. |
| **Studio** | Local web UI at `localhost:4747` via `tutti-ai studio`. |

---

## 2. TypeScript Standards

### Compiler strictness

Every `tsconfig.json` must have — never override:

```json
{
  "strict": true,
  "noUncheckedIndexedAccess": true,
  "exactOptionalPropertyTypes": true,
  "noImplicitReturns": true
}
```

Also enforced: `noUnusedLocals: true`, `noUnusedParameters: true`.
Target: `ES2022`. Module: `ES2022`. Resolution: `bundler`.

### Type safety

- **NEVER** use `any`. Use `unknown` and narrow with type guards or Zod.
- **NEVER** use type assertions (`as X`) without a comment explaining why it is safe.
- **NEVER** use non-null assertions (`!`). Use optional chaining (`?.`) or explicit checks.
- All async functions must have explicit return types.
- Prefer discriminated unions over optional properties.

### Schema validation

- ALL external inputs validated with Zod before use.
- Derive TypeScript types FROM Zod schemas:
  ```typescript
  // Correct:
  const AgentConfigSchema = z.object({ name: z.string(), /* ... */ });
  type AgentConfig = z.infer<typeof AgentConfigSchema>;

  // WRONG:
  interface AgentConfig { name: string; }
  const AgentConfigSchema: z.ZodType<AgentConfig> = z.object({ /* ... */ });
  ```

### Imports


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tuttiai/tutti](https://github.com/tuttiai/tutti) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
