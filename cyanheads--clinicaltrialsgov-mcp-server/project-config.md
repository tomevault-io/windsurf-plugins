---
trigger: always_on
description: **Server:** clinicaltrialsgov-mcp-server
---

# Agent Protocol

**Server:** clinicaltrialsgov-mcp-server
**Version:** 2.4.6
**Framework:** [@cyanheads/mcp-ts-core](https://www.npmjs.com/package/@cyanheads/mcp-ts-core)

> **Read the framework docs first:** `node_modules/@cyanheads/mcp-ts-core/CLAUDE.md` contains the full API reference — builders, Context, error codes, exports, patterns. This file covers server-specific conventions only.

---

## Overview

MCP server wrapping the [ClinicalTrials.gov REST API v2](https://clinicaltrials.gov/data-api/api) — the US National Library of Medicine's registry of ~577K clinical trial studies. Public, read-only, no auth required.

**Design doc:** `docs/design.md` — full MCP surface design, tool schemas, service plan, implementation checklist.
**API reference:** `docs/api-reference.md` — complete ClinicalTrials.gov v2 endpoint reference.

---

## MCP Surface

### Tools (7)

| Name                                   | Description                                                                         |
| :------------------------------------- | :---------------------------------------------------------------------------------- |
| `clinicaltrials_search_studies`        | Search studies with queries, filters, pagination, field selection. Primary tool.    |
| `clinicaltrials_get_study_record`      | Single study by NCT ID. Tool equivalent of the resource for resource-unaware clients. |
| `clinicaltrials_get_study_results`     | Extract outcomes, adverse events, participant flow, baseline for completed studies. |
| `clinicaltrials_get_field_values`      | Discover valid enum values for API fields with study counts.                        |
| `clinicaltrials_get_field_definitions` | Browse the study data model field tree — piece names, types, nesting.               |
| `clinicaltrials_get_study_count`       | Lightweight study count for a query (no data fetched).                              |
| `clinicaltrials_find_eligible`         | Match patient demographics to recruiting trials.                                    |

### Resources (1)

| URI Template               | Description                              |
| :------------------------- | :--------------------------------------- |
| `clinicaltrials://{nctId}` | Single study by NCT ID. Full study data. |

### Prompts (1)

| Name                      | Description                                                  |
| :------------------------ | :----------------------------------------------------------- |
| `analyze_trial_landscape` | Guides multi-step trend analysis using count + search tools. |

---

## What's Next?

When the user asks what to do next, what's left, or needs direction, suggest relevant options based on the current project state:

1. **Add services** — scaffold the `ClinicalTrialsService` using the `add-service` skill
2. **Add tools/resources/prompts** — scaffold definitions using `add-tool`, `add-resource`, `add-prompt` skills
3. **Add tests** — scaffold tests using the `add-test` skill
4. **Field-test definitions** — exercise tools/resources/prompts with real inputs using the `field-test` skill
5. **Run `devcheck`** — lint, format, typecheck, and security audit
6. **Run the `security-pass` skill** — audit handlers for MCP-specific security gaps: output injection, scope blast radius, input sinks, tenant isolation
7. **Run the `tool-defs-analysis` skill** — audit definition language across every tool/resource/prompt: voice, internal leaks, recovery hints, cross-references
8. **Run the `polish-docs-meta` skill** — finalize README, CHANGELOG, metadata for shipping
9. **Run the `maintenance` skill** — sync skills and dependencies after framework updates

Tailor suggestions to what's actually missing or stale — don't recite the full list every time.

---

## Core Rules

- **Logic throws, framework catches.** Tool/resource handlers are pure — throw on failure, no `try/catch`. Plain `Error` is fine; the framework catches, classifies, and formats. Use error factories (`notFound()`, `validationError()`, etc.) when the error code matters.
- **Use `ctx.log`** for request-scoped logging. No `console` calls.
- **Read-only server.** No `ctx.state` needed — the ClinicalTrials.gov API is stateless and public.
- **Secrets in env vars only** — never hardcoded. (This server has no secrets — public API, no auth.)
- **Rate limit awareness.** The API allows ~1 req/sec. Service layer handles retry/backoff.

---

## Patterns

### Tool

```ts
import { tool, z } from "@cyanheads/mcp-ts-core";
import { getClinicalTrialsService } from "@/services/clinical-trials/clinical-trials-service.js";

export const searchStudies = tool("clinicaltrials_search_studies", {
  description: "Search for clinical trial studies from ClinicalTrials.gov.",
  annotations: {
    readOnlyHint: true,
    idempotentHint: true,
    openWorldHint: true,
  },
  input: z.object({
    conditionQuery: z.string().optional().describe("Condition/disease search"),
    pageSize: z
      .number()
      .int()
      .min(1)
      .max(1000)
      .default(10)
      .describe("Results per page"),
  }),
  output: z.object({
    studies: z.array(z.record(z.unknown())).describe("Matching studies"),
    totalCount: z.number().optional().describe("Total matching studies"),
  }),

  async handler(input, ctx) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cyanheads/clinicaltrialsgov-mcp-server](https://github.com/cyanheads/clinicaltrialsgov-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
