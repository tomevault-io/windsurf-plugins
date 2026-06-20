---
trigger: always_on
description: >
---


## Quick Reference

| Action | Command / Method |
|--------|-----------------|
| Start pipeline | Create cron job (see [Activation](#activation)) |
| Stop pipeline | Disable cron job OR set `phase: needs_human` |
| Run one manual cycle | Follow [Cycle Protocol](#cycle-protocol) directly |
| Plan a task | `codex-mcp-call.sh --model gpt-5.2 --sandbox danger-full-access "..."` |
| Implement a task | `codex-mcp-call.sh --model gpt-5.2-codex --full-auto --cwd <project> "..."` |
| Implement (complex) | Same with `--model gpt-5.2-codex-high` |
| Run verification | `./verify.sh` (outputs JSON to stdout) |
| Parse plan output | `python3 extract_plan.py --nonce <NONCE> < raw_output` |
| Parse verdict output | `echo '<json>' \| python3 build_verdict.py --nonce <NONCE> --criteria AC1,AC2,...` |
| LLM verify criterion | `sessions_spawn` sub-agent per acceptance criterion |
| Read state | `yq -r '.<field>' STATE.yaml` |
| Write state | Atomic: flock → yq → temp → mv (see [State Writes](#state-write-protocol)) |
| Post status | `message` action=send to pipeline Discord channel |

## Architecture

### Heartbeat-Driven Execution

```
┌──────────┐  fires every   ┌───────────────┐   dispatch   ┌──────────────┐
│ Cron Job │── 3-5 min ────▶│ Isolated      │─────────────▶│ GPT-5.2      │
│ (driver) │                │ Session       │              │ GPT-5.2-Codex│
└──────────┘                │               │              │ Opus 4.5     │
                            │ flock → read  │              └──────────────┘
┌──────────┐                │ → ONE action  │
│ Discord  │◀── status ────│ → write state │
│ #pipeline│   one-liner   │ → release     │
└──────────┘                └───────────────┘
```

**No ralph.sh. No external loop. Clawdbot's cron IS the heartbeat.**

Each cron tick spawns a fresh isolated session. The session:
1. Acquires `flock` (non-blocking — exits if held)
2. Reads STATE.yaml
3. Runs ONE action
4. Updates STATE.yaml atomically
5. Posts status to Discord
6. Exits (flock released automatically)

STATE.yaml is the continuity. Not the context window.

### Five Actors, Strict Boundaries

| Role | Actor | Writes To |
|------|-------|-----------|
| **Driver** | Clawdbot Cron | Fires sessions on schedule |
| **Orchestrator** | Clawdbot (Claude Opus 4.5) | STATE.yaml (all fields) |
| **Planner** | GPT-5.2 (via Codex MCP) | stdout only (sentinel plan blocks) |
| **Implementer** | gpt-5.2-codex (via Codex MCP) | Source code + git commits |
| **Verifier (Script)** | verify.sh (bash) | stdout only (JSON) |
| **Verifier (LLM)** | Opus 4.5 sub-agent (`sessions_spawn`) | stdout only (sentinel verdict blocks) |

No actor crosses into another's domain. Clawdbot orchestrates but never writes code or judges quality. GPT-5.2 plans but never touches files. gpt-5.2-codex implements but never plans or verifies.

---

## Concurrency Guard

### The Problem

Two cron ticks could read `cycle.status: idle` simultaneously and both claim ownership.

### The Solution: `flock -n`

The ENTIRE cycle is wrapped in a non-blocking filesystem lock:

```bash
(
  flock -n 9 || exit 0   # If locked → another session owns it → exit silently

  # === CRITICAL SECTION ===
  # Read STATE.yaml → check phase → claim → execute → update → release
  # === END ===

) 9>/path/to/project/.deadf/cycle.flock
```

- `flock -n`: Non-blocking. If held, exit immediately. Zero wait, zero conflict.
- Lock released automatically when session exits (even on crash — OS handles it).
- Each project has its own `.deadf/cycle.flock`. Multiple projects don't interfere.

### Stale Lease Recovery

If a session hangs (process alive but stuck), the lease renewal mechanism detects it:

```yaml
# STATE.yaml cycle fields:
cycle:
  status: running
  started_at: "2026-01-30T03:30:00Z"
  session_key: "agent:main:cron:deadfish-mnemo"
  last_heartbeat_at: "2026-01-30T03:35:00Z"
```

**Inside flock, before claiming:**
1. If `cycle.status == running` AND `now - last_heartbeat_at > stale_timeout_min` → recover (reset to idle, log warning)
2. If `cycle.status == running` AND not stale → someone legit is working, but we couldn't get flock... (this case shouldn't happen since we're inside flock — it's a belt-and-suspenders check)

### Lease Renewal

Long-running actions update `last_heartbeat_at` at sub-step boundaries:

```
implement_task:
  1. Update last_heartbeat_at → NOW    (before dispatching to Codex)
  2. Dispatch to gpt-5.2-codex         (may take 20+ min for high reasoning)
  3. Update last_heartbeat_at → NOW    (after Codex returns)
  4. Read git state, update STATE.yaml
```

---

## Cycle Protocol

When triggered (by cron or manual), execute these steps in order:

### Step 0: GUARD

1. Acquire `flock -n` on `<project>/.deadf/cycle.flock`
   - If cannot acquire → exit silently (another session owns the cycle)
2. Read `cycle.status` from STATE.yaml:
   - `idle` → proceed to Step 1
   - `running` + stale (`now - last_heartbeat_at > stale_timeout_min`) → log recovery, reset to `idle`, proceed
   - `running` + not stale → release flock, exit (shouldn't happen inside flock, but safety check)
   - `complete`/`failed` → reset to `idle`, proceed
3. Check `phase`:
   - `needs_human` → post alert to Discord, release flock, exit
   - `complete` → post completion summary, release flock, exit
   - Any other → proceed

### Step 1: LOAD


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Fredasterehub/deadfish-cli](https://github.com/Fredasterehub/deadfish-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
