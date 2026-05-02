---
trigger: always_on
description: This repository is for a Pi coding-agent extension that prunes tool-call trees before the next request is sent.
---

# Project Guidance

This repository is for a Pi coding-agent extension that prunes tool-call trees before the next request is sent.

## Working style
- Keep changes small, focused, and reversible.
- Read existing files before editing them.
- Preserve user work; do not overwrite unrelated changes.
- Prefer Markdown for plans and notes, and keep code and docs aligned.

## Planning
- Use the `planning` skill for any multi-step task.
- Store plans in `.agents/plans/`.
- Use zero-padded numbered plan filenames like `000-first-plan.md`, `001-another-plan.md`, and `002-plan-more.md`.
- Keep plan checklists in sync with actual progress.

## Implementation
- When adding code, include a brief explanation of why the change exists.
- Add tests or a reproducible verification command for behavior changes when possible.

---

## Code Structure

```
pi-context-prune/
├── index.ts                   # Extension entry point — wires all modules together
├── package.json               # Pi package manifest; declares extension at ./index.ts
└── src/
    ├── types.ts               # Shared types, constants, and interfaces (including PruneOn modes)
    ├── config.ts              # Load/save ~/.pi/agent/context-prune/settings.json
    ├── batch-capture.ts       # Serialize turn_end events into CapturedBatch objects
    ├── summarizer.ts          # LLM call that summarizes a CapturedBatch to markdown
    ├── indexer.ts             # Runtime Map<toolCallId, ToolCallRecord> + session persistence
    ├── pruner.ts              # Filter context event messages (removes summarized ToolResultMessages)
    ├── query-tool.ts          # Register the context_tree_query tool for recovering pruned outputs
    ├── stats.ts               # StatsAccumulator for cumulative summarizer token/cost tracking
    └── commands.ts            # /pruner command + interactive settings overlay + summary message renderer
```

### `index.ts` — Extension entry point
Wires all modules together and registers Pi event handlers:
- **`pendingBatches: CapturedBatch[]`** — queue of captured batches not yet summarized; drained by `flushPending`.
- **`flushPending(ctx)`** — summarizes + indexes all pending batches in a **single LLM call** and injects one combined steer message. Sets status to "prune: summarizing…" while working, then restores the status widget with stats (e.g. `prune: ON (Every turn) │ ↑1.2k ↓340 $0.003`). Accumulates summarizer token/cost stats via `StatsAccumulator` and persists them to session. Called immediately on `every-turn` or `agentic-auto`, deferred to the trigger event for other modes.
- **`session_start`** — loads config from `~/.pi/agent/context-prune/settings.json`, rebuilds the in-memory index and stats accumulator, clears `pendingBatches`, updates the footer status widget, and notifies the user of the loaded state.
- **`session_tree`** — rebuilds the index and stats accumulator after branch navigation (pending batches and stats belong to the current branch).
- **`turn_end`** — captures the batch, pushes to `pendingBatches`. Behavior depends on `pruneOn` mode:
  - `every-turn`: flushes immediately.
  - `agent-message`: if the turn has **no** tool results (i.e., a final text-only response), flushes pending batches; otherwise queues.
  - `on-context-tag` / `on-demand`: queues and notifies the user of pending count and trigger.
- **`tool_execution_end`** — when `event.toolName === "context_tag"` and mode is `on-context-tag`, calls `flushPending`.
- **`agent_end`** — safety-net flush for `agent-message` mode: if the agent loop ends before a text-only turn fires (e.g. aborted), flushes any remaining pending batches so they aren't lost.
- **`context`** — filters the message array sent to the LLM, removing `ToolResultMessage` entries that have been summarized. Returns `undefined` (no change) if the index is empty or no messages were removed.

### `src/types.ts` — Shared types and constants
Single source of truth for all interfaces and constants:
- **`CapturedBatch`** / **`CapturedToolCall`** — snapshot of one assistant turn's tool calls + results. `CapturedBatch` also carries `assistantText` (any non-tool-call text from the assistant message).
- **`ToolCallRecord`** — full record stored in the runtime index (includes original `resultText`).
- **`IndexEntryData`** — data shape written to session via `pi.appendEntry` for persistence across restarts.
- **`PruneOn`** — `"every-turn" | "on-context-tag" | "on-demand" | "agent-message" | "agentic-auto"` — when summarization is triggered:
  - `every-turn`: summarize after every tool-calling turn.
  - `on-context-tag`: batch turns, flush when `context_tag` is called.
  - `on-demand`: only when the user runs `/pruner now`.
  - `agent-message`: batch turns, flush when the agent sends a final text-only response (or when the agent loop ends).
  - `agentic-auto`: the LLM decides when to prune by calling the `context_prune` tool, guided by `AGENTIC_AUTO_SYSTEM_PROMPT`.
- **`PRUNE_ON_MODES`** — `{ value, label }` array for interactive selectors.
- **`ContextPruneConfig`** — `{ enabled, summarizerModel, pruneOn }` stored in `~/.pi/agent/context-prune/settings.json`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [championswimmer/pi-context-prune](https://github.com/championswimmer/pi-context-prune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
