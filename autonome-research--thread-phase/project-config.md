---
trigger: always_on
description: Use when the user asks to build a structured agent pipeline, compose multiple agent calls into a deterministic workflow, run agents on a cron/schedule, fan an agent across N items with capped concurrency, chain heterogeneous agents (claude-code → anthropic → codex), persist agent events to a job log, wire memory or session resumption across agent calls, or set up "research loops"/"development loops" with multiple coding agents handing off work. Triggers include phrases like "multi-phase pipeline
---


# thread-phase — agent pipeline orchestration in TypeScript

Three npm packages, all at the same locked version (v3.x):

- **`@autonome-research/thread-phase`** — the framework: `Phase`, `runPipeline`, `runAgentWithTools`, `JobRunner`, `Trigger`, the convenience helpers (`oneShot` / `schedule` / `hook`), plus the `AgentAdapter` protocol under the `/agents` subpath.
- **`@autonome-research/thread-phase-agents`** — adapter implementations for ready agents: `claudeCodeAgent`, `codexCliAgent`, `codexAgent`, `hermesAgent`, `openClawAgent`, `anthropicAgent`, `piAgent`, plus the shared ACP chassis. Heavy SDKs are **optional peer deps** (install only the ones you use).
- **`@autonome-research/thread-phase-cli`** — bin + auto-loader. Single install pulls in the other two.

```bash
# One-command install: gets the full runtime (core + adapters + bin)
npm install -g @autonome-research/thread-phase-cli

# Optional, only if you use that adapter:
# npm install -g @anthropic-ai/sdk @mariozechner/pi-coding-agent openai
```

## When to reach for thread-phase

- The task has 2+ phases that run in a specific order with typed handoff (`fetch → triage → review → compose`)
- One or more phases call an LLM, possibly with tools
- The task is repeatable (cron / systemd / CI) and shouldn't re-derive its plan every run
- Multiple agents collaborate (claude-code does code edits, anthropic synthesizes a report, etc.)
- You want persistent event logs + replay
- You want bounded-concurrency fanout over a list

When **not** to reach for it:
- Single-turn "ask an LLM and print" — use the SDK directly
- A complex DAG with cross-edges — use Temporal/LangGraph/Inngest and embed thread-phase in nodes

## Quickstart — convenience helpers (read this first)

**Default to these helpers for any simple automation.** Reach for the full Phase template (further down) only when the user genuinely needs typed phase composition, multiple steps with shared `ctx`, or `runAgentWithTools` inside a phase.

Each helper returns the default export of a `.thread-phase/pipelines/<name>.ts` file. Drop the file in, then `thread-phase run <name>` or `thread-phase serve`.

```ts
// .thread-phase/pipelines/digest.ts — one-shot script, run on demand
import { oneShot } from '@autonome-research/thread-phase';

export default oneShot(async () => {
  const items = await fetchInbox();
  await sendDigest(await summarize(items));
});
// Run: thread-phase run digest
```

```ts
// .thread-phase/pipelines/morning-digest.ts — scheduled
import { schedule } from '@autonome-research/thread-phase';

export default schedule({ cron: '0 8 * * *' }, async () => {
  // body fires at 8am every day
});
// Or: schedule({ intervalMs: 6 * 60 * 60 * 1000 }, async () => { ... })
// Run: thread-phase serve  (long-running; SIGTERM to stop)
```

```ts
// .thread-phase/pipelines/webhook-digest.ts — HTTP webhook
import { hook } from '@autonome-research/thread-phase';

export default hook({ path: '/digest' }, async (payload, ctx) => {
  await processWebhook(payload);
  return { ok: true };  // becomes the HTTP 200 response body
});
// Run: thread-phase serve  (all hooks share one HTTP server; port 7777 by default)
```

### Decision rule

| User asks for | Reach for |
|---|---|
| "Run X on a schedule" | `schedule({ cron \| intervalMs }, …)` |
| "Build a webhook that does X" | `hook({ path }, …)` |
| "Run this script via thread-phase" | `oneShot(…)` |
| "Pipeline with 2+ phases sharing typed ctx" | `registerPipeline` with Phase template (below) |
| "Heterogeneous agent chain with Thread state" | `registerPipeline` with Phase template (below) |
| "Loop until convergence" | `registerPipeline` + `whileCondition` |
| "Fan an adapter over N items" | `registerPipeline` + `boundedFanoutOf` |

The helpers cover the **first three rows** with one function call. The rest of this doc covers the remaining four.

## Where does X live? Import-path map

Single source of truth. If an import fails, look here first.

| You want… | Import from |
|---|---|
| **Building pipelines**: `Phase`, `runPipeline`, `runPipelineToSummary`, `PipelineCache`, `requireCtx`, `BasePipelineContext`, `PipelineEvent` | `@autonome-research/thread-phase` |
| **First-use helpers**: `oneShot`, `schedule`, `hook`, `CronTrigger`, `HttpTrigger` | `@autonome-research/thread-phase` |
| **Persistence**: `JobRunner`, `SqliteJobStore`, `JobStore`, `JobRecord` | `@autonome-research/thread-phase` |
| **Raw inference loop**: `runAgentWithTools`, `loadInferenceConfig`, `createInferenceClient`, `ToolRegistry` | `@autonome-research/thread-phase` |
| **Triggers**: `TimerTrigger`, `Trigger`, `TriggerEvent`, `runTrigger`, `RunTriggerHandle` | `@autonome-research/thread-phase/triggers` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [autonome-research/thread-phase](https://github.com/autonome-research/thread-phase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
