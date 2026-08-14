---
trigger: always_on
description: Flat single-package repo providing observability for AI coding assistants (MCP server + metrics engine + HTTP proxy). Source lives directly in `src/`. Shared transport/events/pricing code lives in `src/shared/`. All telemetry flows to New Relic.
---

# NR AI Coding Observability: Preflight

Flat single-package repo providing observability for AI coding assistants (MCP server + metrics engine + HTTP proxy). Source lives directly in `src/`. Shared transport/events/pricing code lives in `src/shared/`. All telemetry flows to New Relic.

## Development Commands

```bash
npm run build              # TypeScript build
npm run build:clean        # Clean build output
npm test                   # Run all tests (Jest, maxWorkers: 1)
npm run lint               # ESLint across src/
npm run format             # Prettier (write)
npm run format:check       # Prettier (check only)
```

Build directly:

```bash
npx tsc -b .
```

Run tests for a single file:

```bash
npx jest -- src/metrics/cost-tracker.test.ts
npx jest -- src/shared/harvest/harvest-scheduler.test.ts
```

## Shared Code (`src/shared/`)

**`src/shared/` is a vendored snapshot — do not edit directly.**

**Rules:**

1. **Never edit files under `src/shared/` in this repo.** It is a vendored snapshot. If you find a bug there, open an issue.

## Architecture

### Data Flow (MCP Server — Stdio Mode)

```
Claude Code
  │
  ├─ PreToolUse / PostToolUse hooks
  │    └─> preflight (collector-script.ts)
  │         └─> writes to buffer.jsonl directly (raw fd append)
  │
  └─ MCP stdio connection
       └─> NrMcpServer (server.ts)
            ├─ HookEventProcessor reads buffer.jsonl on poll interval
            │    └─> pairs pre/post → ToolCallRecord
            │         └─> feeds to all metric trackers:
            │              SessionTracker, CostTracker, TaskDetector,
            │              AntiPatternDetector, AuditTrailManager
            │
            ├─ NrIngestManager (HarvestScheduler)
            │    ├─ Events → NR Events API (every 5s)
            │    └─ Metrics → NR Metric API (every 60s)
            │
            └─ MCP Tools (queried by Claude Code)
                 ├─ nr_observe_get_session_stats
                 ├─ nr_observe_get_efficiency_score
                 ├─ nr_observe_get_cost_breakdown
                 ├─ nr_observe_get_anti_patterns
                 ├─ nr_observe_get_recommendations
                 └─ ... (tools listed below)
```

## TypeScript Conventions

### Module System

- ESM throughout (`"type": "module"` in `package.json`)
- `NodeNext` module resolution
- All internal imports use `.js` extensions (required for ESM)
- Strict mode enabled

### Type Patterns

- `interface` for public API contracts and tracker return types
- `type` for unions, intersections, and local aliases
- `readonly` on all interface fields for immutable data shapes
- `Record<string, T>` for dynamic key maps (tool breakdowns, exit code maps)

### Naming

- Files: `kebab-case.ts` (e.g., `session-tracker.ts`, `cost-tracker.test.ts`)
- Classes: `PascalCase` (e.g., `SessionTracker`, `HookEventProcessor`)
- Interfaces: `PascalCase` (e.g., `McpServerConfig`, `FullSessionSummary`)
- Functions: `camelCase` (e.g., `buildSessionSummary`, `parseToolSpecificFields`)
- Constants: `SCREAMING_SNAKE_CASE` (e.g., `DEFAULT_HARVEST_MS`, `TRACKED_METHODS`)
- Test helpers: `camelCase` with `make` prefix (e.g., `makeRecord`, `makeSummary`, `makeManager`)

### Import Order

1. Node.js builtins (`node:fs`, `node:path`, `node:crypto`)
2. External packages (`@modelcontextprotocol/sdk`, `zod`, `commander`)
3. Blank line
4. Shared module imports (`./shared/index.js` or `../shared/index.js`)
5. Local imports (`./types.js`, `../metrics/session-tracker.js`)

### Logger Pattern

Every module creates a scoped logger at module level:

```typescript
import { createLogger } from '../shared/index.js';
const logger = createLogger('module-name');
```

Logger writes to stderr as JSON. Never write to stdout (reserved for MCP stdio transport).

## Metric Tracker Pattern

Trackers in `src/metrics/` do not share one input shape — they fall into four families depending on how data reaches them:

- **Streaming push (void)** — `recordToolCall(record: ToolCallRecord): void`. The majority shape: `SessionTracker`, `TaskDetector`, `WorkflowRunTracker`, `ContextTracker`, `GitEfficiencyTracker`, and most others.
- **Streaming push (signaling)** — `recordToolCall(record: ToolCallRecord): T`, returning a value the caller acts on immediately instead of only accumulating: `TurnTracker` (returns the turn id), `RetryDetector` (returns `ThrashingAlert | null`).
- **Primitive accumulator** — a domain-named method taking specific values instead of a full `ToolCallRecord`, because the input is self-reported (tokens, cost) rather than a raw tool call: `ModelUsageTracker.recordUsage(model, inputTokens, outputTokens, costUsd)`, `CostTracker.recordTokenUsage(usage, model, ctx?)` / `recordEstimatedTokens(...)`, `BudgetTracker.updateCost(...)`.
- **Batch/pull analyzer** — invoked periodically over accumulated history rather than per-call: `AntiPatternDetector.analyze(toolCalls: ToolCallRecord[])`, `EfficiencyScorer.computeScore(task, antiPatterns?)` / `updateScore(...)`.

Whichever family a tracker belongs to, it still:

- Maintains internal state (maps, counters, arrays)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [newrelic-experimental/preflight](https://github.com/newrelic-experimental/preflight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
