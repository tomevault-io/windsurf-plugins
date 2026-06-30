---
trigger: always_on
description: kanbots dispatches one **agent run** per issue, in an isolated git worktree,
---

# Agents

kanbots dispatches one **agent run** per issue, in an isolated git worktree,
backed by either **Claude Code** (`claude -p`) or **Codex** (`codex exec`).
This page describes that lifecycle in detail.

## What runs

Two CLI agents are supported, behind a single
`AgentCliAdapter` interface (`packages/dispatcher/src/`). The adapter
encapsulates argument construction, stream parsing, and decision
plumbing — the rest of the dispatcher doesn't care which CLI is
underneath.

| Agent | Invocation | Notes |
| --- | --- | --- |
| Claude Code | `claude -p` | Reuses your `claude /login` credentials. Best default. |
| Codex | `codex exec` | Requires `codex` on `PATH`. Sign in via "Sign in with codex" or `OPENAI_API_KEY`. Issue drafting and Sentry analysis still run on Claude. |

You pick which one to use per dispatch from the **AI providers** modal:

![AI providers modal showing Claude Code and Codex CLI](assets/agent-cli-picker.png)

Settings → AI providers also sets the workspace default — `claude
(auto)` uses whichever CLI is enabled and signed in.

The Claude flags used are roughly:

```
claude -p \
  --output-format stream-json \
  --verbose \
  --permission-mode bypassPermissions \
  --append-system-prompt "<issue context>" \
  --model <chosen model> \
  [--resume <sessionId>] \
  [--mcp-config <path>]
```

`--permission-mode bypassPermissions` is what makes runs hands-off —
the agent is allowed to use any tool without prompting per call.
Containment is enforced separately (see below) and a pre-push hook in
the worktree prevents network pushes regardless. Codex's equivalent
flags are configured by its adapter in the same place.

## Worktree lifecycle

For each run kanbots:

1. Creates a worktree at `.kanbots/worktrees/issue-<n>-<runId>/`.
2. Branches from the repo's default branch as
   `kanbots/issue-<n>-<runId>`.
3. Stamps an identity file (`.kanbots-identity.json`) used to
   recover orphaned runs.
4. Installs a `pre-push` hook that exits non-zero — any push attempted
   from inside the worktree fails.
5. Spawns the Claude process as a detached process group leader (POSIX)
   so it can be killed cleanly with SIGTERM → SIGKILL escalation.

When the run ends, the worktree stays on disk until you promote it,
discard it, or run another action that removes it. Worktrees and their
branches are first-class kanbots state — you can have many parked runs
across many issues.

## The stream

Both CLIs emit one JSON object per line (Claude's `stream-json` and Codex's
equivalent). The dispatcher's
[`stream-parser`](../packages/dispatcher/src/stream-parser.ts) classifies
each line into one of:

| Event | Meaning |
| --- | --- |
| `text` | Assistant message text |
| `tool_use` | Tool call (Read, Edit, Bash, …) |
| `tool_result` | Tool result body |
| `session` | Session id (used to resume runs) |
| `decision` | Pending decision prompt — the run pauses |
| `result` | End of run, with success/failure and cost |
| `rate_limit` | Provider rate-limit signal — see [cooldown](#rate-limits) |
| `parse_error` | A line we couldn't parse |

Events are persisted into the `agent_events` table and broadcast over the
`agent-runs:events:subscribe` IPC channel. The UI replays from the table
on reconnect, so the live thread survives a renderer reload.

## Decision prompts

![Awaiting decision prompt with four numbered options](assets/run-detail-awaiting-decision.png)

When the agent emits a `decision` event, the dispatcher:

1. Marks the run's card as `awaiting_input`.
2. Stores the question and option list on the card.
3. Stops feeding stdin — the agent process waits.
4. Pushes a `DecisionPayload` (`{ question, options: [{ value, label }] }`)
   to the UI.

You answer it in the issue detail modal. The handler `cards:resolve`
writes the choice back into the agent's stdin and the run continues.
If you close the app mid-decision, the state is on the card — reopen
and respond.

## Containment

Either CLI can technically `Edit` or `Write` to any path the user can
access. kanbots watches every `tool_use` and compares the target path
to the worktree root. Behaviour is governed by **containment mode**:

| Mode | Effect on out-of-worktree edits |
| --- | --- |
| `off` | Allowed, no surface |
| `warn` (default) | Logged as a warning event in the thread |
| `pause` | Run pauses with a decision; you choose to allow or stop |

Set it per-workspace with `containmentMode` in `.kanbots/config.json`, or
per-process via `KANBOTS_CONTAINMENT_MODE=pause` (etc.) in the
environment.

## Cost budgets

Every `result` event carries a USD cost (both CLIs report it). The
dispatcher accumulates it into:

- **Per-run cost** — visible on the run card.
- **Per-session cost** — when run under autopilot.

Two budgets, both optional:

```jsonc
// .kanbots/config.json
{
  "defaults": {
    "runCostBudgetUsd": 2.50,        // single run cap
    "sessionCostBudgetUsd": 25.00    // autopilot session cap
  }
}
```

Setting either to `null` (or omitting it) disables that cap. When a cap
is hit the run is stopped with `stopReason: 'cost-budget'`.

## Rate limits

The streams surface rate-limit hits explicitly. When the dispatcher
detects one it broadcasts a `cooldown:changed` event with the resume

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leodavinci1/kanbots](https://github.com/leodavinci1/kanbots) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
