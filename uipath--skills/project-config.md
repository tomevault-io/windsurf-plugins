---
trigger: always_on
description: > Requires `@uipath/uipath-typescript` **≥ 1.4.1**. Scopes: `Insights Insights.RealTimeData`.
---

# Agents & Agent Memory (Insights RTM) Reference

> Requires `@uipath/uipath-typescript` **≥ 1.4.1**. Scopes: `Insights Insights.RealTimeData`.

Two services, **two different calling conventions** — do not mix them up:

| Service | Subpath | Convention |
|---------|---------|------------|
| `Agents` | `@uipath/uipath-typescript/agents` | **Positional `Date` args**: `getAll(startTime, endTime, options?)` |
| `AgentMemory` | `@uipath/uipath-typescript/agent-memory` | **Options object**: `getTimeline({ startTime?, endTime?, ... })` — dates inside the object |

## Agents Service

```typescript
import { Agents, AgentListSortColumn } from '@uipath/uipath-typescript/agents';
const agents = new Agents(sdk)
```

### getAll(startTime: Date, endTime: Date, options?: AgentListOptions)

The agent list with consumption + health metadata aggregated over the window. Returns `NonPaginatedResponse<AgentListItem>` (or `PaginatedResponse` with pagination options). **Rows are on `.items`.**

`AgentListOptions`: `folderKeys?: string[]`, `agentNames?: string[]`, `projectKeys?: string[]`, `agentId?: string`, `processVersion?: string`, `orderBy?: { column: AgentListSortColumn, desc?: boolean }` + pagination (`pageSize`, `cursor`, `jumpToPage`).

`AgentListSortColumn`: `AgentName`, `ParentProcess`, `LastRun`, `HealthScore`, `LastIncident`, `FolderName`, `QuantityAGU`, `QuantityPLTU`, `FolderPath`.

`AgentListItem` fields: `agentId`, `agentName`, `parentProcess`, `folderKey`, `folderName`, `folderPath`, `lastRun`, `processKey`, `processVersion`, `healthScore` (0–100), `lastIncidentType`, `unitsQuantity`, `unitsName`, `quantityAGU`, `quantityPLTU`. Nullable: `parentProcess`, `folderKey/Name/Path`, `processKey`, `processVersion`, `lastIncidentType`, `unitsName` (may be `null` or `""`).

**Example response** (`.items` — field names exact, values illustrative):

```json
{
  "items": [
    {
      "agentId": "ag-0001", "agentName": "InvoiceTriageAgent",
      "parentProcess": "InvoiceFlow", "folderKey": "f-1001", "folderName": "Finance",
      "folderPath": "Finance", "lastRun": "2026-06-10T18:22:00Z",
      "processKey": "p-0088", "processVersion": "1.2.0",
      "healthScore": 92, "lastIncidentType": null,
      "unitsQuantity": 340, "unitsName": "AGU", "quantityAGU": 340, "quantityPLTU": 0
    },
    {
      "agentId": "ag-0002", "agentName": "ContractReviewAgent",
      "parentProcess": null, "folderKey": "f-1001", "folderName": "Finance",
      "folderPath": "Finance", "lastRun": "2026-06-10T16:05:00Z",
      "processKey": null, "processVersion": null,
      "healthScore": 58, "lastIncidentType": "Error",
      "unitsQuantity": 1210, "unitsName": "AGU", "quantityAGU": 1210, "quantityPLTU": 12
    }
  ],
  "count": 2
}
```

> **Semantics:** `getAll` returns per-agent totals (`quantityAGU`, `healthScore`, `lastIncidentType`) — good for KPIs and ranked tables. For *time-series* (error / latency / consumption trends) use the dedicated timeline methods below — all added in SDK 1.4.1. There is still **no invocation-count timeline** and **no per-percentile method other than `getLatencyTimeline`**.

### getErrors(startTime: Date, endTime: Date, options?: AgentGetErrorsOptions)

Agent error classes (incidents) observed in the window, ranked. Returns `NonPaginatedResponse<AgentError>` (or `PaginatedResponse` with pagination options). **Rows are on `.items`.**

`AgentGetErrorsOptions`: filters (`folderKeys`, `agentNames`, `projectKeys`, `agentId`, `processVersion`) + `orderBy?: { column: AgentErrorSortColumn, desc?: boolean }` + pagination. `AgentErrorSortColumn`: `ExecutionCount`, `ErrorTitle`, `Type`, … (import from `@uipath/uipath-typescript/agents`).

`AgentError` fields: `type`, `description`, `agentId`, `agentName`, `jobKey`, `parentProcess`, `firstSeen`, `folderKey`, `folderName`, `folderPath`, `count`, `firstSeenJob`, `lastSeenJob`.

```json
{ "items": [
  { "type": "ToolError", "description": "Tool 'search' timed out", "agentId": "ag-0002", "agentName": "ContractReviewAgent", "count": 14, "firstSeen": "2026-06-02T09:11:00Z", "folderName": "Finance" }
], "count": 1 }
```

### getErrorsTimeline(startTime: Date, endTime: Date, options?)

Time-series of error counts grouped by agent. Returns a **bare array** `[{ name, value, date }]` — `name` is the agent name, `value` the error count, `date` the bucket. Options: filters + `limit?` (top-N agents, default 10).

```json
[ { "name": "ContractReviewAgent", "value": 6, "date": "2026-06-02" },
  { "name": "InvoiceTriageAgent", "value": 1, "date": "2026-06-02" } ]
```

### getConsumptionTimeline(startTime: Date, endTime: Date, options?)

Time-series of AGU consumption. Returns a **bare array** `[{ timeSlice, aguConsumption }]` — native chart shape. Options: filters.

```json
[ { "timeSlice": "2026-06-01T00:00:00Z", "aguConsumption": 120 },
  { "timeSlice": "2026-06-02T00:00:00Z", "aguConsumption": 340 } ]
```

### getLatencyTimeline(startTime: Date, endTime: Date, options?)

Time-series of agent latency per percentile. Returns a **bare array** `[{ name, value, date }]` — `name` is the percentile (`"P50"` / `"P95"`), `value` is **milliseconds**, `date` the bucket. Options: filters.

```json
[ { "name": "P50", "value": 820, "date": "2026-06-02" },

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [UiPath/skills](https://github.com/UiPath/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
