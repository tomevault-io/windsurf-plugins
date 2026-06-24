---
trigger: always_on
description: You are working with reharness, a finite state machine framework for multi-agent pipelines. It orchestrates Pi coding agents as states in an FSM with typed transitions, guards, and events.
---

# reharness — LLM Reference (runtime API usage)

You are working with reharness, a finite state machine framework for multi-agent pipelines. It orchestrates Pi coding agents as states in an FSM with typed transitions, guards, and events.

This file is the **how-to-write-it API reference**. For the *why* — the formal execution model, the static analyzer, and the data-flow model — see **`.claude/theory/`** (runtime.md, analysis.md, pipeline.md). The runtime is a deterministic hierarchical Moore-action transducer with run-to-completion: each state runs to completion and emits one event; transitions are total and fail loud; composite states (parallel/loop/call) are run-to-completion sub-computations.

## Core Concepts

**Pipeline** — a finite state machine. States execute entry actions and emit events. Events trigger transitions to the next state. Guards conditionally select transitions. Final states end the pipeline.

**State** — has an `entry` action (async function) and transitions (`on`). Entry returns an event name (string) or void (= `DONE` event).

**Transition** — maps event to target state. Can have guards (conditions). Array of targets = first matching guard wins.

**Agent** — a Pi coding agent subprocess. Gets a markdown prompt (`.md`) and a task string. Runs autonomously with tools (read/write/edit/bash/grep/find).

**Command** — user-facing entry point in `reharness/commands/`. Parses arguments, constructs a pipeline, returns it.

## Project Structure

The `reharness/` bundle is a first-class, liftable deliverable; regenerable run-exhaust lives under a hidden `.cache/`.

```
project/
└── reharness/                # the deliverable (version it / ship it / `mv` it — self-contained)
    ├── skeletons/     # <id>.xml — source of truth for generated pipelines
    ├── prds/          # <cmdId>.md — approved intent archive
    ├── commands/      # Each file = one slash command, auto-discovered (codegen output for generated ones)
    ├── skills/        # <topic>.md — domain-skills from research (grounded knowledge, attached to leaves)
    ├── agents/        # Per command: <command>/<name>/SYSTEM.md (+ optional harness.json)
    ├── lib/           # Code-state implementations (<id>-states.ts)
    └── .cache/        # run-exhaust (gitignored): runs/ (run-*/{state.json, work/, trace/}), evolve/ (ledger), scratch/
```

## Writing a Pipeline

```typescript
import { definePipeline } from 'reharness';

definePipeline({
  config: { slug, idea },
  initial: 'plan',           // Start state (must exist)
  agents: ctx.agents,        // Optional, defaults to reharness/agents/

  states: {
    // Linear state — do work, move on
    plan: {
      entry: async (ctx) => { await ctx.agent('planner', 'Plan the project'); },
      on: 'code',  // shorthand: DONE event → 'code'
    },

    // Branching state — entry returns event name
    verify: {
      entry: async (ctx) => {
        const ok = await ctx.shell('npx tsc --noEmit', 'tsc');
        return ok ? 'PASS' : 'FAIL';  // event name
      },
      on: {
        PASS: 'done',
        FAIL: [
          { target: 'fix', guard: (ctx) => ctx.retries('v') < 3 },
          { target: 'error' },  // no guard = fallback
        ],
      },
    },

    // Fix + retry loop
    fix: {
      entry: async (ctx) => {
        ctx.retry('v');  // increment counter
        await ctx.agent('fixer', 'Fix the errors');
      },
      on: 'verify',  // back to verify
    },

    // Final states — pipeline ends here
    done:  { type: 'final', status: 'success', entry: async (ctx) => { ctx.emit('Done!'); } },
    error: { type: 'final', status: 'error' },
  },
});
```

### State Rules

1. `initial` state must exist in `states`.
2. Every transition target must reference an existing state. Validated at definition time — typos throw immediately.
3. At least one `{ type: 'final' }` state required.
4. Entry returns `string` → that string is the event. Entry returns `void` → event is `'DONE'`.
5. `on: 'target'` is shorthand for `on: { DONE: 'target' }`.
6. Guard arrays: evaluated in order, first with `guard === undefined` or `guard() === true` wins.

### Transition Formats

```typescript
// Simple: always go to target
on: 'nextState'

// Event map: different events → different targets
on: {
  PASS: 'success',
  FAIL: 'error',
}

// Guarded: first matching guard wins
on: {
  FAIL: [
    { target: 'fix', guard: (ctx) => ctx.retries('k') < 3 },
    { target: 'error' },  // fallback
  ],
}

// Single guarded transition
on: {
  DONE: { target: 'next', guard: (ctx) => someCondition },
}
```

## Composite states (run-to-completion sub-machines)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bes-dev/reharness](https://github.com/bes-dev/reharness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
