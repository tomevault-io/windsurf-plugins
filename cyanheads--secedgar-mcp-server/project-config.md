---
trigger: always_on
description: **Server:** secedgar-mcp-server
---

# Agent Protocol

**Server:** secedgar-mcp-server
**Version:** 0.15.11
**Framework:** [@cyanheads/mcp-ts-core](https://www.npmjs.com/package/@cyanheads/mcp-ts-core) `^0.13.11`
**Engines:** Bun ≥1.4.0, Node ≥24.0.0
**MCP SDK:** `@modelcontextprotocol/server` ^2.2.0
**Zod:** ^4.6.5

Query SEC EDGAR filings, XBRL financials, and company data through MCP. Read-only, no API keys required. Full design: `docs/sec-edgar-mcp-design.md`.

> **Read the framework docs first:** `node_modules/@cyanheads/mcp-ts-core/CLAUDE.md` contains the full API reference — builders, Context, error codes, exports, patterns. This file covers server-specific conventions only.

---

## Core Rules

- **Logic throws, framework catches.** Tool/resource handlers are pure — throw on failure, no `try/catch`. Plain `Error` is fine; the framework catches, classifies, and formats. Use error factories (`notFound()`, `validationError()`, etc.) when the error code matters.
- **Use `ctx.log`** for request-scoped logging. No `console` calls. Attach operation-specific fields with `withExtra(ctx, { … })` — `RequestContext` is closed, so a bare `{ ...ctx, field }` spread no longer type-checks.
- **Use `ctx.state`** for tenant-scoped storage. Never access persistence directly.
- **Ask for missing input** with `return ctx.requestInput(...)`, reading the answers back through `ctx.inputs` when the handler is re-entered. Never await a caller mid-handler.
- **Secrets in env vars only** — never hardcoded.
- **Cut noise.** Add only what earns its place: no speculative generality, no guards for states the framework already prevents (Zod-validated params, classified errors), no abstraction until a third caller proves it, no option nothing sets.
- **Close the loop on issues.** When implementing work tracked by a GitHub issue, comment on the issue with what landed and close it. Do both — a comment without a close leaves stale issues open; a close without a comment leaves no record of what shipped. The comment is for future readers — state the concrete changes, not the conversation that produced them.

---

## What's Next?

When the user asks what's next or needs direction, suggest options based on the current project state. Common next steps:

1. **Add tools/resources/prompts** — scaffold new definitions using the `add-tool`, `add-app-tool`, `add-resource`, `add-prompt` skills
2. **Add services** — scaffold domain service integrations using the `add-service` skill
3. **Add tests** — scaffold tests for existing definitions using the `add-test` skill
4. **Field-test definitions** — exercise tools/resources/prompts with real inputs using the `field-test` skill, get a report of issues and pain points
5. **Run the `security-pass` skill** — audit handlers for MCP-specific security gaps: output injection, scope blast radius, input sinks, tenant isolation
6. **Run the `polish-docs-meta` skill** — finalize README, CHANGELOG, metadata, and agent protocol for shipping
7. **Run the `maintenance` skill** — investigate changelogs, adopt upstream changes, and sync skills after `bun update --latest`

Tailor suggestions to what's actually missing or stale — don't recite the full list every time.

---

## MCP Surface

### Tools

| Name | Description | Key Inputs |
|:-----|:------------|:-----------|
| `secedgar_company_search` | Find companies and retrieve entity info with optional recent filings | `query`, `include_filings?`, `forms?`, `filing_limit?`, `filed_after?`, `filed_before?` |
| `secedgar_search_filings` | Search EDGAR filings since 1993 — full-text (2001+) plus archive-backed browse for pre-2001 ranges | `query?`, `forms?`, `filed_after?`, `filed_before?`, `limit?` |
| `secedgar_get_filing` | Fetch a specific filing's metadata and document content | `accession_number`, `cik?`, `content_limit?`, `document?` |
| `secedgar_get_financials` | Get historical XBRL financial data for a company | `company`, `concept`, `taxonomy?`, `period_type?`, `unit?`, `limit?` |
| `secedgar_get_snapshot` | One-call financial profile — latest value of every supported concept from a single companyfacts read | `company`, `taxonomy?`, `period_type?` |
| `secedgar_get_material_events` | 8-K filings with item codes decoded, filterable by item | `company`, `items?`, `filed_after?`, `filed_before?`, `limit?` |
| `secedgar_get_insider_transactions` | Form 4 / 4-A insider transactions parsed from ownership XML | `company`, `transaction_type?`, `limit?`, `filed_after?`, `filed_before?` |
| `secedgar_get_institutional_holdings` | 13F-HR quarterly institutional holdings parsed from the information table | `company`, `quarter?`, `limit?`, `offset?`, `consolidate?` |
| `secedgar_find_holders` | Reverse 13F lookup — which institutional managers reported holding an issuer | `issuer`, `cusip?`, `quarter?`, `limit?` |
| `secedgar_get_beneficial_owners` | 5%+ blockholders of an issuer from structured SCHEDULE 13D/13G XML | `issuer`, `form_kind?`, `include_amendments?`, `limit?` |
| `secedgar_get_fund_holdings` | Fund portfolio holdings from the quarterly NPORT-P report | `fund`, `series_id?`, `report_date?`, `limit?`, `offset?` |
| `secedgar_fetch_frames` | Fetch SEC XBRL frames for one concept × one period across all reporting companies | `concept`, `period`, `taxonomy?`, `unit?`, `limit?`, `offset?`, `sort?` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cyanheads/secedgar-mcp-server](https://github.com/cyanheads/secedgar-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
