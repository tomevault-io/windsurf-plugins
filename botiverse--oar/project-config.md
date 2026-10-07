---
trigger: always_on
description: Mapping claims follow current OAR source. Observations name their binary: the
---

# Claude Code

Mapping claims follow current OAR source. Observations name their binary: the
live contract ([`experiments/live-contract.ts claude`](../../experiments/live-contract.ts))
ran on **claude 2.1.268** (darwin arm64, haiku, 2026-09-11); the probes on
**2.1.237** and **2.1.261** (linux x64) are listed in the
[experiments index](../../experiments/README.md); later observations carry
their version inline. The latest daily check re-ran `basic`, `tool-detail` and
`resume` on **2.1.289** (2026-10-04,
[report](../../experiments/runtime-version-checks/2026-10-04.md)). Versions
are evidence baselines, not a support range.
Tags follow the [spec conventions](../spec/README.md): `[src]` vendor source,
`[sym]` binary symbols, `[env]` observed. See the [runtime index](README.md)
for status labels.

## Native concepts and calling interfaces

Claude Code owns the agent loop, tools, context management, and persistent
conversation. A **session** is persistent conversation identity; a **user turn**
can contain multiple **model steps** and tool executions. Assistant messages
carry text, thinking, and tool-use blocks; tool results return in user-message
blocks. A result ends a user turn, not the conversation or necessarily every
native child. [Agent loop][native-loop], [streaming input][native-input].

Native **subagents** have separate conversations and can run concurrently.
The `Agent` tool invocation and `parent_tool_use_id` identify child activity;
child identity, tool-call identity, and main session identity are different
concepts. [Subagents][native-subagents].

Programs have two entry points:

- **CLI print mode:** `claude -p` supports structured output and bidirectional
  `stream-json`; a long-lived process accepts multiple user turns. OAR uses
  this interface. [CLI reference][native-cli].
- **Claude Agent SDK:** TypeScript `query()` exposes an async message stream;
  SDK APIs add configuration, callbacks, session discovery, history
  retrieval, and fork. OAR does not use the SDK.
  [SDK sessions][native-sessions], [permissions][native-permissions].

## High-level mapping to OAR

| Native concept or interface | OAR mapping (record stream) |
|---|---|
| CLI process | One owned subprocess per OAR Session; stdio carries inputs, controls, and frames. Its exit is an `exited` response record (answering `dispose` when OAR caused it). |
| Persistent session ID | `Session.id`, passed as `--session-id` or `--resume`. Every record carries it as `sessionId`. |
| stream-json frame | One `Frame` record per stdout line (control traffic below and the effort read-back at open are the exceptions): `type` = `type[/subtype]`, `native` = the frame verbatim, `events` = OAR's readings (text_delta, reasoning, tool_call_started/ended, user_message, turn_ended, usage, model, task_*, compaction_ended). Frames OAR does not interpret (`rate_limit_event`, `system/thinking_tokens`, …) carry no events. No `spanId`: claude frames carry no turn id. |
| User turn and `result` | The `prompt` request record starts the turn; the `result` frame ends it with a `turn_ended` event (`aborted` while OAR's own interrupt is outstanding, `failed` on `is_error`, else `completed`) plus a `usage` event. |
| Subagent messages (`parent_tool_use_id`) | `agentPath = [...parentPath, taskCallId]` ([details](#observation-children-and-history)); `capabilities.attribution` is `attributed`. |
| `tool_use` / `tool_result` blocks | `tool_call_started` (`callId`, `tool`, `input`) and `tool_call_ended` (`callId`, `content`, `result`): `is_error: true` is `failed`; `false` or an absent field is `ok` ([evidence](#tool-call-outcome-reporting)). |
| `control_request` / `control_response` | OAR's interrupt is an `abort` request record whose id is the `control_request` id; claude's `control_response` becomes its `accepted`/`rejected` response. A `control_request` from claude is recorded as a Frame plus an unanswered `toApp` request (none arrive under `--dangerously-skip-permissions`); `events()` reads it as `app_request` with the request subtype as `type`. |
| `system/task_*` | `task_started`, `task_updated`, `task_ended` events for commands, subagents and backgrounded MCP calls (claude moves a main-conversation MCP call past two minutes to the background). `background_tasks_changed` (the live set) and `task_progress` carry no events. |
| `system/compact_boundary` | The after-the-fact compaction report: a `compaction_ended` event, outcome `completed`, `trigger` from `compact_metadata.trigger` (`manual` \| `auto`). The frame carries `compact_metadata { trigger, pre_tokens, post_tokens?, cumulative_dropped_tokens? }` [sym 2.1.272]. claude has no start frame, so no `compaction_started`, no `retry` (401s are retried silently) and no `tool_call_progress` (tool output arrives whole in the `user` tool_result frame). |
| SDK configuration and interaction APIs | Only `--model`, `--effort` (confirmed by `get_settings` at open) and the system prompt flags. |

Sources: [adapter](../../packages/oar/src/runtimes/claude/session.ts),
[projection](../../packages/oar/src/runtimes/claude/projection.ts),
[task frames](../../packages/oar/src/runtimes/claude/tasks.ts),
[Session contract](../../packages/oar/src/contracts/session.ts).

## Capability details


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [botiverse/oar](https://github.com/botiverse/oar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
