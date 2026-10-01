---
trigger: always_on
description: Async multi-step research, list-building, enrichment, and structured extraction via `POST /agent/runs`.
---

# Agent API Reference

Async multi-step research, list-building, enrichment, and structured extraction via `POST /agent/runs`.

## Canonical Docs Links

- Agent API guide: `https://exa.ai/docs/reference/agent-api-guide`
- Create a run: `https://exa.ai/docs/reference/agent-api/create-a-run`
- Get a run: `https://exa.ai/docs/reference/agent-api/get-a-run`
- List runs: `https://exa.ai/docs/reference/agent-api/list-runs`
- List run events: `https://exa.ai/docs/reference/agent-api/list-run-events`
- Cancel a run: `https://exa.ai/docs/reference/agent-api/cancel-a-run`
- Delete a run: `https://exa.ai/docs/reference/agent-api/delete-a-run`
- Exa Connect overview: `https://exa.ai/docs/reference/agent-api/connect/overview`
- Connect combining providers: `https://exa.ai/docs/reference/agent-api/connect/combining-providers`
- Connect providers: `https://exa.ai/docs/reference/agent-api/connect/fiber`, `https://exa.ai/docs/reference/agent-api/connect/similarweb`, `https://exa.ai/docs/reference/agent-api/connect/baselayer`, `https://exa.ai/docs/reference/agent-api/connect/affiliatecom`, `https://exa.ai/docs/reference/agent-api/connect/particle`, `https://exa.ai/docs/reference/agent-api/connect/financialdatasets`, `https://exa.ai/docs/reference/agent-api/connect/jinko`, `https://exa.ai/docs/reference/agent-api/connect/additional-partners`

## Overview

Use `/agent` when a workflow needs more than a single search or extraction call:

- build lists from open-ended criteria, then enrich each result
- research entities across many fields with citations
- run multi-hop tasks such as "find companies, then find decision makers"
- produce structured JSON from a long-running web research task
- continue from a previous run with a follow-up request

For simpler low-latency retrieval, prefer `/search`.

## Request Shape

```json
POST https://api.exa.ai/agent/runs
{
  "query": "Find engineering leaders at AI infrastructure companies that raised a Series A or B in the last 6 months.",
  "effort": "auto",
  "outputSchema": {
    "type": "object",
    "properties": {
      "people": {
        "type": "array",
        "maxItems": 10,
        "items": {
          "type": "object",
          "properties": {
            "name": { "type": "string" },
            "job_title": { "type": "string" },
            "linkedin_url": { "type": "string", "format": "uri" }
          },
          "required": ["name", "job_title", "linkedin_url"]
        }
      }
    },
    "required": ["people"]
  }
}
```

## Core Fields

| Field | Type | Notes |
| --- | --- | --- |
| `query` | string | Required natural-language task |
| `input.data` | object[] | Existing rows to research or enrich |
| `input.exclusion` | object[] | Records or entities Agent should avoid surfacing |
| `outputSchema` | object | JSON Schema for validated `output.structured` |
| `previousRunId` | string | Continue from a completed prior run |
| `effort` | string | Always set explicitly: `minimal`, `low`, `medium`, `high`, `xhigh`, or `auto`. |
| `budget.maxCostDollars` | number | Per-run spend ceiling in dollars, `1` to `100`. Only accepted with `auto`; defaults to `$5`. |
| `dataSources` | object[] | Exa Connect providers to attach to the run, for example `{ "provider": "similarweb" }` |

`outputSchema` supports JSON Schema. Bound list outputs with `maxItems` where possible so output size and enrichment cost are predictable.

Always send an explicit `effort`. Prefer `auto` unless the task or product needs a fixed cost/latency band (`low` for cheap/fast, `high` / `xhigh` for harder research).

To request contact information, describe the desired contact fields in the schema. Use standard JSON Schema formats such as `{ "type": "string", "format": "email" }`, `{ "type": "string", "format": "phone" }`, and `{ "type": "string", "format": "uri" }`.

`auto` is metered by usage and capped by `budget.maxCostDollars` (default `$5`). The cap is a ceiling, not a fixed price: runs that finish early cost less. Fixed efforts bill a flat per-request price and reject `budget`.

## Lifecycle

Create returns a run object immediately; final output is available only after a terminal status. Every integration must complete this cycle; do not stop at create:

1. Create a run with `POST /agent/runs`.
2. Save the returned `id`, which has the `agent_run_` prefix.
3. Wait for the run to reach a terminal status (`completed`, `failed`, or `cancelled`), either by:
   - **Polling:** `GET /agent/runs/{id}` until `status` is terminal, or
   - **SSE events:** `Accept: text/event-stream` on create, or replay from `GET /agent/runs/{id}/events` with `Last-Event-ID`.
   The two are equivalent; pick one per integration.
4. Check how the run ended: on `completed`, read `output`; on `failed` or `cancelled`, surface the error to the caller or UI. Only `completed` carries results — reading `output` without checking the status makes failures look like empty successes, and a hand-rolled wait loop that exits only on `completed` never finishes for failed runs.

Completed runs include:

- `output.text`: natural-language answer or summary
- `output.structured`: validated JSON matching `outputSchema`, when provided
- `output.grounding`: citations for text or structured fields

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [exa-labs/agent-skills](https://github.com/exa-labs/agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
