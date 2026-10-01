---
trigger: always_on
description: handles it.
---

# Sub-agents

> Status: **implemented** (`src/core/agents/`). Talon's own delegation
> mechanism: any chat turn, on any backend, can spawn an isolated agent with
> its own backend and model, talk to it while it runs, and be woken with its
> report.

This is deliberately **not** the Claude SDK's sub-agent feature. Talon owns
the mechanism so it works identically on `claude`, `codex`, `kilo`,
`opencode` — anything with a `background` capability — and so a chat on one
backend can delegate to an agent on another.

## The model

A **sub-agent** is one isolated one-shot run (`runOneShotAgent`) that some
other agent work started. It has:

- an **id** (`agt_<8 hex>`) and a content-free **label**;
- a **brief** — the entire context it gets, because it has no conversation
  history and no shared scratchpad;
- a **parent**, which is either a chat or another sub-agent;
- its own **backend and model**, defaulting to the parent's backend and that
  backend's default model;
- a **mailbox** its parent can put instructions in;
- a hard **timeout**, a **task-table** entry, and a per-run markdown log at
  `~/.talon/workspace/logs/agents/<id>.md`.

Spawning returns immediately with the id. The parent keeps working; the
report arrives later, through the wake turn (chat parent) or the mailbox
(agent parent). That is the shape Claude Code's background agents have, and
the reason it is worth having: delegation that does not block the delegator
and does not flood its context.

Three modules, one concern each:

| Module        | Owns                                                                 |
| ------------- | -------------------------------------------------------------------- |
| `registry.ts` | identity, the lifecycle state machine, mailboxes, parent/child edges |
| `runner.ts`   | backend/model resolution, the isolated run, settlement, kills        |
| `delivery.ts` | getting text between an agent and its parent, in both directions     |

`context.ts` is a dependency-free leaf holding the `agent:<id>` vocabulary —
imported directly by `backend/claude-sdk` and the gateway, because routing
that knowledge through `index.ts` would drag the runner (and the backend
pool) into their import graphs.

## Lifecycle

```
queued → running → done | failed | killed | timed_out
```

`queued` lasts only as long as it takes to acquire the backend and resolve
the model; a spawn that fails there is discarded without trace (nothing ran,
so there is nothing to report on) and the tool returns the reason.

**Result precedence**, applied at settlement:

1. `report_result` — the agent's own summary/details. This is the channel.
2. the run's **last assistant text**, captured through
   `OneShotAgentParams.onAssistantText`, when the agent finished without
   reporting.
3. neither → the run settles `failed`. A sub-agent that says nothing has not
   done its job, and a parent is always told so.

A result reported before a kill or a timeout is kept: the parent gets the
partial answer plus the terminal state. **Every** terminal state is
delivered — silence is never an outcome.

When an agent settles, its own still-running children are killed. Their
reports would have nowhere to go, so leaving them running only spends tokens.

## Communication

| Direction             | Tool             | Mechanism                                           |
| --------------------- | ---------------- | --------------------------------------------------- |
| agent → parent        | `report_result`  | settles the run; delivered on settlement            |
| agent → parent        | `message_parent` | interim note, delivered immediately, run continues  |
| parent → agent        | `send_to_agent`  | bounded FIFO mailbox (32), drained by `check_inbox` |
| agent → its own inbox | `check_inbox`    | drains everything waiting, exactly once             |

A **chat parent** is woken with a synthetic turn —
`execute({ source: "agent", senderName: "Agent", prompt: "[System: AGENT
FINISHED …]" })` — exactly as a trigger fires one. The turn resumes the
chat's own session, so the model reads the report with full conversational
context and decides for itself whether the user hears about it.

An **agent parent** gets a mailbox push. If it has already settled, the
message is logged and dropped: reviving a finished run to hear late news is
worse than losing the news. A mailbox push past the cap is _refused_, not
silently dropped, so `send_to_agent` can tell the parent it did not land.

`wait_for_agent` exists for the case where the very next thing the parent
does depends on the answer. It is bounded (≤120s, with a 180s bridge budget
above it) so an MCP tool-call timeout can never trip, and it is **not** the
completion channel — the wake turn is, and it fires whether or not anyone
is waiting.

## Tools

Parent-side (available in every chat, and inside an `agent:*` run):

- `spawn_agent({ brief, label, backend?, model?, effort?, timeout_s?, preflight? })` →
  `{ agent_id, backend, model }` — `preflight` see [Pre-flight lane](#pre-flight-lane)
- `list_agents()` — this chat's agents, descendants included
- `agent_status({ agent_id })` — everything but the brief
- `wait_for_agent({ agent_id, timeout_s ≤ 120 })`
- `send_to_agent({ agent_id, text })`
- `kill_agent({ agent_id })`

Agent-side (refused anywhere but an `agent:*` context):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thefalconry/talon](https://github.com/thefalconry/talon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
