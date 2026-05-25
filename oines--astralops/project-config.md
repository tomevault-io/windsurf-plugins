---
trigger: always_on
description: Do not rely on chat context for the AstralOps event contract. Treat this file as the project-level source of truth.
---

# AstralOps Agent Notes

## Event contract memory

Do not rely on chat context for the AstralOps event contract. Treat this file as the project-level source of truth.

AstralOps normalized event families:

```text
session.*
turn.*
message.*
reasoning.*
tool.*
approval.*
ask.*
plan.*
queue.*
workspace.*
memory.*
subagent.*
hook.*
control.*
```

UI states and event meanings that must be first-class:

```text
idle / running / requires_action / reconnecting / failed
assistant streaming
reasoning block
plan mode block
command output
file diff approval
Ask/user input form
MCP elicitation form/url
prompt queued / dequeued / cancelled
rate limit
compact boundary
projection hydrated / pushed
SSH degraded / reconnected
```

Codex raw event references observed from prior sessions:

```text
session_meta
turn_context
task_started
user_message
agent_message
token_count
turn_aborted
response_item.message
response_item.function_call
response_item.function_call_output
response_item.custom_tool_call
response_item.custom_tool_call_output
patch_apply_end
```

Implementation rule:

```text
Claude/Codex raw events must be preserved in AstralEvent.raw.
UI and AstralOps business logic must consume AstralEvent.kind + AstralEvent.normalized.
Do not invent ad hoc event families outside the normalized list above without updating this file and protocol docs/tests.
Do not implement speculative fallback mappings for Claude/Codex events.
When adding or changing event normalization/rendering, first capture real local Claude Code/Codex samples into fixtures and implement against those exact observed shapes.
If an event is not covered by a fixture, preserve it only as hidden control.raw for debug/replay. Do not create generic visible UI for it, and do not map it into a semantic event until a real fixture proves the shape.
Do not add "best guess" UI branches for event names that have not been observed locally.
```

Architecture and fallback rule:

```text
Do not add speculative fallback logic, broad catch-all mappings, or redundant defensive branches to make an uncertain case appear handled.
Every behavior mapping, permission response, state transition, and UI surface must be backed by a real Claude/Codex fixture, source-backed protocol shape, or an explicit rule in this file.
If a real issue points to an architectural mismatch, stop and identify the architectural fix or refactor boundary for user confirmation instead of layering another patch on top of the mismatch.
Prefer deleting or narrowing unsupported branches over preserving "just in case" behavior.
Temporary compatibility code is allowed only when tied to a specific observed version/shape and documented with the fixture or source evidence that requires it.
```

Real-task validation priority:

```text
Real Claude/Codex validation must prioritize user-visible task flow failures over isolated event rendering.
The highest-priority failures are repeated Ask/plan/permission loops, non-resumable confirmations presented as resumable, stale pending interactions after a turn has already failed or completed, tasks stuck in requires_action with no valid next action, and agents continuing to ask for the same missing permission after the user has accepted, declined, skipped, or cancelled.
Every local/SSH and default/full-permission test scenario must record whether the agent made forward progress, stopped correctly, or entered a loop/stuck state. A scenario is not passing just because the UI rendered the latest event.
If a real task exposes repeated questions, repeated plan confirmations, repeated permission prompts, or a mismatch between the displayed action and the actual agent continuation semantics, treat it as a blocker before expanding coverage to lower-risk event types.
```

UI implementation rule:

```text
Visible UI copy must not use emoji or decorative Unicode symbols. Use plain text and lucide icons for affordances/status. Keyboard hints must be plain labels such as Enter, Cmd+Enter, or ESC.
Permission, command, file-change, Ask, MCP elicitation, and plan confirmation surfaces must show the concrete decision target from AstralEvent.normalized, such as command, cwd, tool name, reason, file/change summary, prompt, or params. Do not show generic approval text when normalized data contains a more specific target.
```

## Current event coverage audit

Last audited: 2026-05-23.

Claude Code local currently uses `claude -p --output-format stream-json --verbose --include-partial-messages --include-hook-events`; it is not yet a full Claude SDK/control-protocol host. This means stdout stream-json is covered, but control requests/responses/cancel requests and most hook lifecycle events are not yet implemented.

Claude Code covered from real fixtures:

```text
system -> session.native
assistant text/partial text -> message.delta
assistant thinking/partial thinking -> reasoning.delta
assistant tool_use TodoWrite -> tool.todo
assistant tool_use AskUserQuestion -> ask.requested
assistant tool_use ExitPlanMode -> plan.updated
assistant tool_use Read/LS/Glob/Grep/WebSearch/Write/Edit/MultiEdit/Bash -> tool.started with category
user tool_result -> tool.completed
result.permission_denials ExitPlanMode -> approval.requested(kind=plan)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oines/AstralOps](https://github.com/oines/AstralOps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-25 -->
