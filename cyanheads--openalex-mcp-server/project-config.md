---
trigger: always_on
description: **Server:** openalex-mcp-server
---

# Agent Protocol

**Server:** openalex-mcp-server
**Version:** 0.7.2
**Framework:** [@cyanheads/mcp-ts-core](https://www.npmjs.com/package/@cyanheads/mcp-ts-core) `^0.10.9`
**Engines:** Bun ≥1.3.0, Node ≥24.0.0

> **Read the framework docs first:** `node_modules/@cyanheads/mcp-ts-core/CLAUDE.md` contains the full API reference — builders, Context, error codes, exports, patterns. This file covers server-specific conventions only.

---

## Domain

[OpenAlex](https://openalex.org) is a fully open catalog of the global research system — 270M+ works, 90M+ authors, 100K+ sources. CC0 data, no API key required (email address optional for polite pool access).

**Entity types:** works, authors, sources, institutions, topics, keywords, publishers, funders. All share a uniform API (list/filter, search, get-by-ID, group-by, autocomplete).

**Critical workflow:** Names are ambiguous, IDs are not. Always resolve names to IDs first via `openalex_resolve_name` before using them in filters.

**Reference docs:** `README.md` covers the current tool and prompt surface; `docs/tree.md` shows the current repository layout.

### MCP Surface

| Type | Name | Purpose |
|:-----|:-----|:--------|
| Tool | `openalex_search_entities` | Search, filter, sort, or retrieve by ID |
| Tool | `openalex_analyze_trends` | Group-by aggregation for trends/distributions |
| Tool | `openalex_resolve_name` | Name-to-ID resolution via autocomplete |
| Tool | `openalex_get_citation_graph` | One-hop citation graph traversal (cites/cited_by/related_to) |
| Tool | `openalex_describe_fields` | List valid filter/group_by/select field names per entity type |
| Prompt | `openalex_literature_review` | Guided systematic literature search workflow |
| Prompt | `openalex_research_landscape` | Quantitative research landscape analysis |

No resources — entity lookups need `select` for payload control, which fits tools better than URI templates.

### Config

| Env Var | Required | Description |
|:--------|:---------|:------------|
| `OPENALEX_API_KEY` | No | Email for the OpenAlex polite pool (10x faster rate limits); omit for anonymous access |
| `OPENALEX_BASE_URL` | No | Default: `https://api.openalex.org` |

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
import { getOpenAlexService } from '@/services/openalex/openalex-service.js';
import { ENTITY_TYPES } from '@/services/openalex/types.js';

export const resolveNameTool = tool('openalex_resolve_name', {
  description: 'Resolve a name or partial name to an OpenAlex ID. Returns up to 10 matches with disambiguation hints.',
  annotations: { readOnlyHint: true, openWorldHint: true },
  input: z.object({
    entity_type: z.enum(ENTITY_TYPES).optional().describe('Entity type to search. Omit for cross-entity search.'),
    query: z.string().describe('Name or partial name to resolve.'),
  }),
  output: z.object({
    results: z.array(z.object({
      id: z.string().describe('OpenAlex ID.'),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cyanheads/openalex-mcp-server](https://github.com/cyanheads/openalex-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-05 -->
