---
trigger: always_on
description: **Server:** ntfy-mcp-server
---

# Developer Protocol

**Server:** ntfy-mcp-server
**Version:** 2.0.1
**Framework:** [@cyanheads/mcp-ts-core](https://www.npmjs.com/package/@cyanheads/mcp-ts-core) `^0.9.1`
**Engines:** Bun ≥1.3.0, Node ≥24.0.0
**MCP SDK:** `@modelcontextprotocol/sdk` ^1.29.0
**Zod:** ^4.4.3

> **Read the framework docs first:** `node_modules/@cyanheads/mcp-ts-core/CLAUDE.md` contains the full API reference — builders, Context, error codes, exports, patterns. This file covers server-specific conventions only.
>
> **ntfy upstream API docs:** mirrored under `docs/ntfy/` — `publish.md`, `subscribe/api.md`, `emojis.md`, `examples.md`, `index.md`. See `docs/ntfy/SOURCES.md` for the pinned commit and refresh steps.

---

## What's Next?

When the user asks what to do next, what's left, or needs direction, you can suggest relevant options based on the current project state. Some common next steps:

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

- **Logic throws, framework catches.** Tool/resource handlers are pure — throw on failure, no `try/catch` for control flow. The narrow `try/catch` blocks in this codebase exist solely to translate upstream ntfy errors into typed contract failures via `ctx.fail()` before re-throwing; everything else bubbles for framework auto-classification. Use error factories (`notFound()`, `forbidden()`, `validationError()`, …) when no contract entry fits.
- **Use `ctx.log`** for request-scoped logging. No `console` calls.
- **Auth scope is the configured `NTFY_BASE_URL`.** When a tool's `base_url` argument differs from the configured base, `NtfyService` strips the auth header before sending — never widen this to "always forward credentials" without explicit operator opt-in.
- **Secrets in env vars only** — never hardcoded. `NTFY_AUTH_TOKEN` is mutually exclusive with `NTFY_AUTH_USERNAME` / `NTFY_AUTH_PASSWORD`; the basic-auth pair must be set together. Validation enforces this at config load.
- **Treat topic names as secrets.** Anyone who knows a topic name can publish or subscribe — surface that in tool descriptions and never log full topic names at info level when the topic is private.

---

## Patterns

### Tool

`ntfy_search_emoji_tags` — minimal in-memory tool, illustrates the basic shape:

```ts
import { tool, z } from '@cyanheads/mcp-ts-core';
import { getEmojiTagService } from '@/services/emoji-tags/emoji-tag-service.js';

export const ntfySearchEmojiTags = tool('ntfy_search_emoji_tags', {
  description:
    "Look up ntfy emoji tag short codes. Use the returned `tag` strings in `ntfy_publish_message`'s `tags` field…",
  annotations: { readOnlyHint: true, openWorldHint: false },
  input: z.object({
    query: z.string().optional().describe('Substring to match (case-insensitive).'),
    limit: z.number().int().positive().max(200).default(25).describe('Max matches.'),
  }),
  output: z.object({
    matches: z.array(z.object({
      tag: z.string().describe('Short code.'),
      emoji: z.string().describe('Rendered Unicode emoji.'),
    })).describe('Tag → emoji rows.'),
    total: z.number().describe('Total matches before truncation.'),
    truncated: z.boolean().describe('True when more matches existed than `limit`.'),
  }),

  handler(input) {
    return getEmojiTagService().search(input.query, input.limit);
  },

  format: (result) => [{
    type: 'text',
    text: result.matches.length === 0
      ? `No emoji tags matched (total: ${result.total}).`
      : `${result.matches.map(m => `| \`${m.tag}\` | ${m.emoji} |`).join('\n')}`,
  }],
});
```

`ntfy_publish_message` — typed error contract with upstream classification:

```ts
import { tool, z } from '@cyanheads/mcp-ts-core';
import { JsonRpcErrorCode, validationError } from '@cyanheads/mcp-ts-core/errors';
import { getNtfyService } from '@/services/ntfy/ntfy-service.js';

export const ntfyPublishMessage = tool('ntfy_publish_message', {
  description: 'Send or update a push notification on an ntfy topic…',
  annotations: { openWorldHint: true },
  input: /* … */,
  output: /* … */,

  errors: [

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cyanheads/ntfy-mcp-server](https://github.com/cyanheads/ntfy-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
