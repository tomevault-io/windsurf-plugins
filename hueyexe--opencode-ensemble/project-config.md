---
trigger: always_on
description: opencode-ensemble is an OpenCode plugin that enables agent teams: multiple
---

# opencode-ensemble — Agent Guidelines

## What This Is

opencode-ensemble is an OpenCode plugin that enables agent teams: multiple
agents running in parallel with peer-to-peer communication, shared task
management, and coordinated execution. Built entirely on the public OpenCode
plugin SDK (@opencode-ai/plugin) with zero internal dependencies.

## Architecture

### Plugin SDK Constraint

This is a plugin, not a core contribution. We only use APIs from
@opencode-ai/plugin and @opencode-ai/sdk. No access to OpenCode internals
(Storage, Bus, Lock, SessionPrompt, etc.).

Key SDK primitives:
- client.session.create() — create teammate sessions
- client.session.promptAsync() — inject messages + auto-wake (fire-and-forget)
- client.session.abort() — cancel/shutdown teammates
- client.session.status() — poll session idle/busy state
- event hook — subscribe to session.status events for state transitions
- tool hook — register the 14 team tools
- tool.execute.before hook — rate limiting + sub-agent isolation

### Storage

SQLite via bun:sqlite (zero dependencies). Four tables:
- team — team config (name, lead session, status, delegate mode)
- team_member — member registry (name, session ID, agent, status)
- team_task — shared task board (content, status, priority, assignee, deps)
- team_message — message log (from, to, content, delivered flag)

WAL mode. Migrations via PRAGMA user_version.

### Message Delivery

All messages delivered via client.session.promptAsync(). Single atomic
operation: injects user message + starts prompt loop if idle. No polling,
no file watching, no custom pub/sub.

### State Machines

Two-level per member:
- Member status: ready | busy | shutdown_requested | shutdown | error
- Execution status: idle | starting | running | cancel_requested |
  cancelling | cancelled | completing | completed | failed | timed_out

Driven by session.status events from the plugin event hook.

### Sub-Agent Isolation

Enforced via tool.execute.before hook. Maintains a Map<sessionID, parentSessionID>
populated from session events. When a team tool call arrives from an unknown
session, walks the parent chain (max depth 10). If any ancestor is a team
member, the call is blocked. This covers sub-agents at arbitrary depth.

## The 14 Tools

| Tool                | Who Can Use | Purpose                              |
|---------------------|-------------|--------------------------------------|
| team_create         | Any session | Create a new team, caller is lead    |
| team_spawn          | Lead only   | Spawn a teammate with a prompt (supports plan_approval mode) |
| team_message        | Any member  | Send message to teammate or lead (approve/reject plans) |
| team_broadcast      | Any member  | Send message to all team members     |
| team_tasks_list     | Any member  | View the shared team task board      |
| team_tasks_add      | Any member  | Add tasks to the shared board        |
| team_tasks_complete | Any member  | Mark a task complete, unblock deps   |
| team_claim          | Any member  | Atomically claim a pending task      |
| team_results        | Any member  | Retrieve full message content        |
| team_shutdown       | Lead only   | Request teammate shutdown, preserves branch |
| team_merge          | Lead only   | Merge a shutdown teammate's branch   |
| team_cleanup        | Lead only   | Archive team, safety-net merge remaining |
| team_status         | Any member  | View members, statuses, task summary |
| team_view           | Any member  | Navigate TUI to teammate's session   |

## Hooks

Three hooks wired in index.ts:

- `experimental.chat.system.transform` — injects team state into the lead's
  system prompt (member statuses, task counts, anti-polling guidance). Injects
  a short role reminder for teammates.
- `experimental.session.compacting` — preserves team context during session
  compaction so the model remembers it's leading/part of a team after long
  conversations are compressed.
- `shell.env` — sets ENSEMBLE_TEAM, ENSEMBLE_MEMBER, ENSEMBLE_ROLE, and
  ENSEMBLE_BRANCH in teammate shells.

## Settled Decisions (Do Not Re-Debate)

1. SQLite via bun:sqlite — not file JSON, not in-memory-only
2. promptAsync for message delivery — not session injection, not polling
3. 14 separate tools — not a unified action tool, no exceptions
4. Fire-and-forget spawn — not blocking, not tmux
5. tool.execute.before for rate limiting — token bucket, in-memory
6. tool.execute.before for sub-agent isolation — full descendant tracking via parent chain
7. Worktree isolation on by default — each teammate gets their own git
   worktree via client.worktree.create(). Opt out with worktree: false
   for read-only agents. Lead merges branches after cleanup.
8. Plan approval is prompt-enforced, not permission-based — the teammate's
   context message tells it to send a plan and wait for approval. No tool-level
   gating.
9. Graceful shutdown with manual force — team_shutdown requests graceful stop
   by default. Pass force: true to abort immediately.
10. Never await promptAsync — all promptAsync calls are fire-and-forget.
    Awaiting blocks the caller if the transport is slow or broken. Messages
    are persisted in the DB first; the idle-flush backstop handles delivery.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hueyexe/opencode-ensemble](https://github.com/hueyexe/opencode-ensemble) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
