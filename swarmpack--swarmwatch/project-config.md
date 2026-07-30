---
trigger: always_on
description: This doc is the **canonical** reference for how SwarmWatch integrates with **Claude Code** hooks.
---

# SwarmWatch Claude Code Hooks

This doc is the **canonical** reference for how SwarmWatch integrates with **Claude Code** hooks.

Official Claude Code hooks reference (for future updates):
- https://code.claude.com/docs/en/hooks

---

## 0) Architecture (end-to-end)

SwarmWatch uses the same architecture for Claude Code as it does for Cursor:

```text
Claude Code Hook
  → spawns swarmwatch-runner (child process)
  → runner reads JSON from stdin
  → runner POSTs normalized events to SwarmWatch local control plane
      http://127.0.0.1:4100/event
  → control plane broadcasts events + approvals to the Tauri UI over WebSocket
      ws://127.0.0.1:4100
  → (if control hook) runner MAY wait for a decision (bounded)
  → runner prints stdout JSON expected by Claude Code for that hook
  → runner exits (Claude Code continues)
```

**Key implementation files in this repo:**

- Runner entrypoint: `src-tauri/src/bin/swarmwatch-runner.rs`
- Claude adapter: `src-tauri/src/runner/adapters/claude.rs`
- Local control plane (HTTP + WS): `src-tauri/src/control_plane.rs`
- UI websocket client + activity log: `src/useAgentStates.ts`
- UI renderer: `src/App.tsx`

---

## 1) Scope: hooks we use

Claude Code supports many hook events. SwarmWatch intentionally uses only:

- `UserPromptSubmit`
- `PreToolUse` *(PRIMARY CONTROL HOOK)*
- `PostToolUse`
- `PostToolUseFailure`
- `Stop`
- `SessionEnd`

Everything else is ignored for now.

---

## 1.1 Common fields (present in every Claude hook payload)

Claude sends a JSON payload to the hook handler’s **stdin**. These common fields are the ones
SwarmWatch cares about (others may exist and are ignored):

| Field | Description |
|---|---|
| `session_id` | Current session identifier (used as `agentInstanceId`) |
| `transcript_path` | Path to conversation JSON |
| `cwd` | Current working directory when the hook is invoked |
| `permission_mode` | Current permission mode: `default\|plan\|acceptEdits\|dontAsk\|bypassPermissions` |
| `hook_event_name` | Name of the event that fired |

Tool events (like `PreToolUse`) also include:

- `tool_name`
- `tool_input` (tool-specific payload)

---

## 2) Identity model

- `agentFamily = "claude"`
- `agentInstanceId = session_id` (one avatar per Claude session)
- control plane normalizes `agentKey = ${agentFamily}:${agentInstanceId}`

> **Edge case:** if `session_id` is missing, the current adapter defaults to `"default"`,
> which collapses all Claude sessions into a single avatar.

---

## 3) Approval semantics (allow / deny / ask)

Claude’s control hook (`PreToolUse`) supports three decisions:

- `allow` — proceed
- `deny` — block
- `ask` — delegate to Claude’s native UX (“ask me in Claude”)

SwarmWatch UI policy:

- Allow / Deny are decided in the overlay.
- For **Ask**, the overlay should present a third button: **“Decide in Claude”**.
  - Meaning: do not decide in SwarmWatch; let Claude show its native permission UI.
  - Claude will display `permissionDecisionReason` inside Claude Code.

**Timeout policy:** if SwarmWatch does not receive a decision in time, we default to `ask`
so Claude can show its built-in permission UI.

---

## 3.1 Tool groups (Claude) → SwarmWatch policy

SwarmWatch classifies Claude tools into three buckets:

### READING (auto-allow)
- `Read`, `Glob`, `Grep`, `LS`, `WebSearch`, `WebFetch`

### EDITING (auto-allow; optionally show diff)
- `Edit`, `Write`, `NotebookEdit`

### APPROVAL REQUIRED (show Approve / Deny / Decide in Claude)
- `Bash`
- `Task`
- any `mcp__*` tool

---

## 4) State mapping (Claude → SwarmWatch)

SwarmWatch’s normalized UI states are the same as Cursor (see `docs/Cursor.md`).

For Claude Code, we treat tool usage as:

- `PreToolUse` → `awaiting` (while waiting for approval)
- after decision:
  - allow → `running`
  - deny → `error` (unified bad-outcome state; UI label should show **Denied**)
  - ask → **remain in `awaiting`** (Decide in Claude)

Then:
- `PostToolUse` → `thinking`
- `PostToolUseFailure` → `error`
- `Stop` → `done` or `error`
- `SessionEnd` → `inactive`

---

## 4.1) Inactive semantics (critical)

`inactive` is a **dangerous** state in SwarmWatch.

Canonical meaning: the session is over and the avatar can be removed.

SwarmWatch reaches `inactive` for exactly two reasons:

1) **Explicit session end hook**: Claude `SessionEnd` → `inactive`
2) **UI inactivity timeout** (planned policy): if no new events are received for a session for **90 seconds**, the overlay marks it `inactive`.

Notes:
- `Stop` is **not** a session end signal. `Stop` maps to `done|error`.
- The UI may also offer a per-session **× dismiss** control for `inactive` avatars (planned). This is not persisted and does not affect Claude Code itself.

> Note: keep `docs/statecycle.md` as the canonical, repo-wide state machine reference.

---

## 5) Hook: UserPromptSubmit

**Meaning (Claude docs):** fires when you submit a prompt, before Claude processes it.

**SwarmWatch UI state:** `thinking`.

### stdin (example)

Claude’s stdin contains a rich session context. SwarmWatch primarily uses:

```jsonc
{
  "hook_event_name": "UserPromptSubmit",
  "session_id": "sess_01",
  "prompt": "Summarize src/App.tsx",
  "cwd": "/Users/me/project",
  "...": "additional Claude fields"
}
```

### stdout


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SwarmPack/SwarmWatch](https://github.com/SwarmPack/SwarmWatch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
