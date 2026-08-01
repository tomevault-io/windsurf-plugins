---
trigger: always_on
description: **Server:** {{PACKAGE_NAME}}
---

# Developer Protocol

**Server:** {{PACKAGE_NAME}}
**Version:** 0.1.0
**Framework:** [@cyanheads/mcp-ts-core](https://www.npmjs.com/package/@cyanheads/mcp-ts-core) `^{{FRAMEWORK_VERSION}}`
**Engines:** Bun ≥1.3.0, Node ≥24.0.0
**MCP SDK:** `@modelcontextprotocol/sdk` {{MCP_SDK_VERSION}}
**Zod:** {{ZOD_VERSION}}

> **Read the framework docs first:** `node_modules/@cyanheads/mcp-ts-core/CLAUDE.md` contains the full API reference — builders, Context, error codes, exports, patterns. This file covers server-specific conventions only.

---

## First Session

This project was just scaffolded with `bunx @cyanheads/mcp-ts-core init`. You're holding a production-grade MCP framework with the hard parts already solved — error handling, telemetry, auth, transport, validation, lifecycle. What's missing is the **domain**. Your job: design the tool, resource, and service surface with the user, then implement it as small pure handlers that throw — the framework catches, classifies, and instruments the rest. Design before code; the user's first messages set direction, so wait for them before scaffolding definitions.

> **Remove this section** from CLAUDE.md / AGENTS.md after completing these steps. The skills and conventions below remain — this block is one-time onboarding only.

1. **Get your bearings.** Take stock of the project tree, the skills in `skills/`, and the tools/MCP servers available. Light tool use is fine for context-building — you're mapping the territory, not committing yet.
2. **Read the framework docs** — `node_modules/@cyanheads/mcp-ts-core/CLAUDE.md` (builders, Context, errors, exports, conventions)
3. **Run the `setup` skill** — read `skills/setup/SKILL.md` and follow its checklist (project orientation, agent protocol file selection, echo definition cleanup, skill sync)
4. **Design the server** — read `skills/design-mcp-server/SKILL.md` and work through it with the user to map the domain into tools, resources, and services before scaffolding

---

## What's Next?

When the user asks what's next or needs direction, suggest options based on the current project state. Common next steps:

1. **Re-run the `setup` skill** — ensures CLAUDE.md, skills, structure, and metadata are populated and up to date with the current codebase
2. **Run the `design-mcp-server` skill** — if the tool/resource surface hasn't been mapped yet, work through domain design
3. **Add tools/resources/prompts** — scaffold new definitions using the `add-tool`, `add-app-tool`, `add-resource`, `add-prompt` skills
4. **Add services** — scaffold domain service integrations using the `add-service` skill
5. **Add tests** — scaffold tests for existing definitions using the `add-test` skill
6. **Field-test definitions** — exercise tools/resources/prompts with real inputs using the `field-test` skill, get a report of issues and pain points
7. **Run `devcheck`** — lint, format, typecheck, and security audit
8. **Run the `security-pass` skill** — audit handlers for MCP-specific security gaps: output injection, scope blast radius, input sinks, tenant isolation
9. **Run the `polish-docs-meta` skill** — finalize README, CHANGELOG, metadata, and agent protocol for shipping
10. **Run the `maintenance` skill** — investigate changelogs, adopt upstream changes, and sync skills after `bun update --latest`

Tailor suggestions to what's actually missing or stale — don't recite the full list every time.

---

## Core Rules

- **Logic throws, framework catches.** Tool/resource handlers are pure — throw on failure, no `try/catch`. Plain `Error` is fine; the framework catches, classifies, and formats. Use error factories (`notFound()`, `validationError()`, etc.) when the error code matters.
- **Use `ctx.log`** for request-scoped logging. No `console` calls.
- **Use `ctx.state`** for tenant-scoped storage. Never access persistence directly.
- **Check `ctx.elicit`** for presence before calling.
- **Secrets in env vars only** — never hardcoded.
- **Close the loop on issues.** When implementing work tracked by a GitHub issue, comment on the issue with what landed and close it. Do both — a comment without a close leaves stale issues open; a close without a comment leaves no record of what shipped. The comment is for future readers — state the concrete changes, not the conversation that produced them.

---

## Patterns

### Tool

```ts
import { tool, z } from '@cyanheads/mcp-ts-core';

export const searchItems = tool('search_items', {
  description: 'Search inventory items by query.',
  annotations: { readOnlyHint: true },
  input: z.object({
    query: z.string().describe('Search terms'),
    limit: z.number().default(10).describe('Max results'),
  }),
  output: z.object({
    items: z.array(z.object({
      id: z.string().describe('Item ID'),
      name: z.string().describe('Item name'),
    })).describe('Matching items'),
  }),
  auth: ['inventory:read'],

  async handler(input, ctx) {
    const items = await findItems(input.query, input.limit);
    ctx.log.info('Search completed', { query: input.query, count: items.length });
    return { items };
  },

  // format() populates content[] — the markdown twin of structuredContent.
  // Different clients read different surfaces (Claude Code → structuredContent,
  // Claude Desktop → content[]); both must carry the same data.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cyanheads/mcp-ts-core](https://github.com/cyanheads/mcp-ts-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
