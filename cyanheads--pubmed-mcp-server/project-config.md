---
trigger: always_on
description: **Server:** @cyanheads/pubmed-mcp-server
---

# Agent Protocol

**Server:** @cyanheads/pubmed-mcp-server
**Version:** 2.6.6
**Framework:** [@cyanheads/mcp-ts-core](https://www.npmjs.com/package/@cyanheads/mcp-ts-core)

> **Read the framework docs first:** `node_modules/@cyanheads/mcp-ts-core/CLAUDE.md` contains the full API reference — builders, Context, error codes, exports, patterns. This file covers server-specific conventions only.

---

## What's Next?

When the user asks what to do next, what's left, or needs direction, suggest relevant options based on the current project state:

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
- **Check `ctx.elicit` / `ctx.sample`** for presence before calling.
- **Secrets in env vars only** — never hardcoded.

---

## Patterns

### Tool

```ts
import { tool, z } from '@cyanheads/mcp-ts-core';
import { getNcbiService } from '@/services/ncbi/ncbi-service.js';

export const spellCheckTool = tool('pubmed_spell_check', {
  description: "Spell-check a query and get NCBI's suggested correction.",
  annotations: { readOnlyHint: true, openWorldHint: true },

  input: z.object({
    query: z.string().min(2).describe('PubMed search query to spell-check'),
  }),

  output: z.object({
    original: z.string().describe('Original query'),
    corrected: z.string().describe('Corrected query (same as original if no suggestion)'),
    hasSuggestion: z.boolean().describe('Whether NCBI suggested a correction'),
  }),

  async handler(input, ctx) {
    ctx.log.info('Executing pubmed_spell_check tool', { query: input.query });
    const result = await getNcbiService().eSpell({ db: 'pubmed', term: input.query });
    return { original: result.original, corrected: result.corrected, hasSuggestion: result.hasSuggestion };
  },

  format: (result) => {
    if (result.hasSuggestion) {
      return [{ type: 'text', text: `**Suggested correction:** "${result.corrected}" (original: "${result.original}")` }];
    }
    return [{ type: 'text', text: `No spelling corrections suggested for: "${result.original}"` }];
  },
});
```

### Resource

```ts
import { resource, z } from '@cyanheads/mcp-ts-core';
import { getNcbiService } from '@/services/ncbi/ncbi-service.js';

export const databaseInfoResource = resource('pubmed://database/info', {
  name: 'database-info',
  title: 'PubMed Database Info',
  description: 'PubMed database metadata including field list, last update date, and record count.',
  mimeType: 'application/json',
  params: z.object({}),
  output: OutputSchema,

  async handler(_params, ctx) {
    ctx.log.info('Fetching PubMed database info');
    const raw = await getNcbiService().eInfo({ db: 'pubmed' });
    // ... parse XML response ...
    return { dbName, description, count, lastUpdate, fields };
  },

  list: () => ({
    resources: [{ uri: 'pubmed://database/info', name: 'PubMed Database Info' }],
  }),
});
```

### Server config

```ts
// src/config/server-config.ts — lazy-parsed, separate from framework config
import { z } from '@cyanheads/mcp-ts-core';
import { parseEnvConfig } from '@cyanheads/mcp-ts-core/config';

const emptyAsUndefined = (v: unknown) => (v === '' ? undefined : v);

const ServerConfigSchema = z.object({
  apiKey: z.preprocess(emptyAsUndefined, z.string().optional()).describe('NCBI API key'),
  toolIdentifier: z.string().default('pubmed-mcp-server').describe('NCBI tool identifier'),
  adminEmail: z.preprocess(emptyAsUndefined, z.email().optional()).describe('Admin contact email'),
  requestDelayMs: z.coerce.number().min(50).max(5000).default(334).describe('Request delay in ms'),
  maxRetries: z.coerce.number().min(0).max(10).default(6).describe('Max retry attempts'),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cyanheads/pubmed-mcp-server](https://github.com/cyanheads/pubmed-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
