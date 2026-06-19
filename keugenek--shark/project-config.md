---
trigger: always_on
description: > **STOP — check ARGUMENTS first.** Look at the ARGUMENTS line at the bottom of this file. If the first word matches a sub-command below, execute ONLY that sub-command and stop. Do NOT apply the Shark Pattern.
---


# 🦈 The Shark Pattern

> **STOP — check ARGUMENTS first.** Look at the ARGUMENTS line at the bottom of this file. If the first word matches a sub-command below, execute ONLY that sub-command and stop. Do NOT apply the Shark Pattern.
>
> | First word | Action |
> |------------|--------|
> | `loop` | **Run the shell script.** Strip "loop" from args to get the task. Run: `bash "<skill_dir>/shark.sh" "<task>"` (where `<skill_dir>` is the "Base directory for this skill" shown above). Pass `--max-loops N` as `SHARK_MAX_LOOPS=N` env var, `--timeout S` as `SHARK_LOOP_TIMEOUT=S`. Defaults: 50 loops, 25s timeout. **Do not apply the Shark Pattern manually.** |
> | `status` | Read `<skill_dir>/shark-exec/state/pending.json`, check `.shark-done` and `SHARK_LOG.md`. Report state or "No active shark jobs." |
> | `clean` | Remove `.shark-done`, `SHARK_LOG.md`, `shark-exec/state/pending.json`. Report what was cleaned. |
> | `autotune` | Read `<skill_dir>/state/timings.jsonl`, compute stats, recommend settings. |
> | `help` | List sub-commands and summarize the Shark Pattern briefly. |
>
> **If ARGUMENTS does not start with a sub-command above**, treat the full text as a task and apply the Shark Pattern below.

> *A shark that stops swimming dies. An agent that waits for tools wastes compute.*

**Works with:** Claude Code · Codex · Gemini CLI · Cursor · Windsurf · Aider · OpenClaw · any LLM agent

## When to Use This Skill

Trigger this skill when the user says:
- "use the shark pattern"
- "non-blocking agent"
- "never wait for tools"
- "spawn background workers"
- "parallel subagents"
- "keep the main agent moving"
- or when you notice you're about to block on a slow tool (web fetch, build, test run, API call)

## The Rule

**Every LLM turn must complete in under 30 seconds.**

If any operation would take longer:
1. Spawn a remora (`sessions_spawn` with `mode: "run"`)
2. Continue reasoning immediately
3. Incorporate remora results when they arrive

You are **never** in I/O wait. You are **always** reasoning about something.

## Lifecycle

```
┌─────────────┐
│  DECOMPOSE  │  Break task into N independent subtasks
└──────┬──────┘
       │ spawn N remoras (+ 1 pilot fish when first completes early)
       ▼
┌─────────────┐
│    SPAWN    │  sessions_spawn × N, all parallel, record session IDs
└──────┬──────┘
       │ main agent keeps reasoning (never waits)
       ▼
┌─────────────┐     timeout/crash
│   MONITOR   │ ──────────────────► MARK ⏱/❌ (partial still useful)
└──────┬──────┘
       │ all done OR deadline hit
       ▼
┌─────────────┐
│  AGGREGATE  │  Collect results, note failures, merge pilot fish draft
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   REPORT    │  Single coherent response with failure count noted
└─────────────┘
```

**No nested remoras.** If a remora is running, it executes inline — remoras cannot spawn their own remoras. Only the main shark spawns.

## The Pattern

### Bad (Ralph-style blocking):
```
think → call slow tool → WAIT 60s → think → call slow tool → WAIT 45s → ...
```

### Good (Shark-style non-blocking):
```
think → spawn remora(slow tool) → think about something else
     → spawn remora(another tool) → synthesize partial results
     → receive remora result → incorporate → swim on
```

## Implementation

When applying the Shark Pattern, structure your work like this:

### 1. Identify blocking operations
Before calling any tool, ask: "Will this take more than 20-30 seconds?"

Slow tools (always spawn — these are examples, not things this skill executes):
- Web searches / page fetches
- Remote commands
- Build / test / CI runs
- File system scans over large directories
- API calls with unknown latency
- LLM inference calls (coding agents)

Fast tools (run inline, never spawn):
- Reading local files
- Simple calculations
- String manipulation
- Memory lookups

### 2. Spawn remoras

```
sessions_spawn({
  task: "Do the slow thing and return the result",
  mode: "run",
  runtime: "subagent",
  streamTo: "parent"  // optional: stream output back
})
```

Spawn multiple remoras in parallel when possible — don't serialize unless there's a data dependency.

### 3. Keep the main fin moving

After spawning, immediately continue:
- Plan the next step
- Work on a different part of the task
- Summarize what you know so far
- Prepare to incorporate results

### 4. Incorporate results

When remora results arrive, weave them in and continue. Never re-do work a remora already completed.

If your runtime keeps subagents alive after completion, close them once you've incorporated their result. In Codex that means: wait for the remora, use its output, then `close_agent(id)` unless you intentionally plan to reuse that same agent.

## Timing Budget

| Operation | Budget | Action |
|-----------|--------|--------|
| File read | < 2s | Inline |
| Web search | 5-30s | Spawn |
| Remote command | 10-120s | Spawn |
| Build/test | 30-300s | Spawn |
| Coding agent | 60-600s | Spawn |
| Memory search | < 3s | Inline |

## Example: Multi-Step Research Task

**Without Shark (blocking):**
```
1. Search web for X        [wait 15s]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [keugenek/shark](https://github.com/keugenek/shark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
