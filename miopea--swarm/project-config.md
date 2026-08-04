---
trigger: always_on
description: > See `~/.claude/CLAUDE.md` for universal rules (design principles, code quality, TDD workflow, quality gates).
---

# Swarm — Project Guide

> See `~/.claude/CLAUDE.md` for universal rules (design principles, code quality, TDD workflow, quality gates).

## 1. Quick Reference

### Essential Rules
| Rule | Action |
|------|--------|
| Before commit | Use `/commit` slash command |
| Pre-commit validation | Use `/check` slash command |
| Bug fix | Use `/fix-and-ship` or `/diagnose` first |
| Test failures | STOP — fix before continuing |
| Warnings | STOP — warnings = failures |
| `type: ignore` | FORBIDDEN — fix the type error |
| Creating a file | SEARCH existing code first |
| Installed tool stale? | `uv tool uninstall swarm-ai && uv cache clean swarm-ai && uv tool install --no-cache .` |

### Key Files
| File | When to Check |
|------|---------------|
| `swarm.yaml` | Configuring workers, drones, queen, groups |
| `src/swarm/drones/state_tracker.py` | Debugging state detection issues (provider patterns in `src/swarm/providers/`) |
| `src/swarm/drones/pilot.py` | Understanding the poll loop and drone actions |
| `src/swarm/server/daemon.py` | Core daemon lifecycle, events, WebSocket broadcasts |
| `src/swarm/server/api.py` | All HTTP/WebSocket endpoints |
| `src/swarm/web/templates/dashboard.html` | Dashboard UI and JS |

---

## 2. What This Is

A Python web tool for orchestrating multiple Claude Code agents.
Workers run in PTYs managed by a pty-holder sidecar. The background drones handle routine decisions.
The Queen (headless `claude -p`) handles complex decisions.

### Autonomous task momentum

Swarm **pushes** work into worker PTYs — workers don't need to poll for assignments.
Four mechanisms keep momentum without operator intervention (the first three
landed together as task #225; the fourth was added in task #250):

1. **Task-push dispatch on assignment.** `swarm_create_task(target_worker=X)`
   routes through `daemon.assign_and_start_task()` by default, which injects the
   task description straight into X's PTY within one poll cycle. Pass
   `start=False` to queue without dispatch (for Queen/operator staging). Self-
   targeted tasks (caller == target) never dispatch — no interleaving with the
   caller's own turn.
2. **Idle-watcher drone.** A periodic sweep (`drones/idle_watcher.py`,
   `DroneConfig.idle_nudge_interval_seconds`, default 180 s) nudges RESTING /
   SLEEPING workers that have an ASSIGNED / ACTIVE task. 15-minute debounce
   per (worker, task) keeps a stuck worker from being spammed. Every nudge logs
   as `AUTO_NUDGE` under `LogCategory.DRONE`.
3. **Post-ship self-loop.** When `daemon.complete_task()` ships a task, it
   fires `start_task()` for the next ASSIGNED task belonging to the same worker
   (lowest number first). ACTIVE follow-ups are skipped — they're already
   running somewhere. Empty queues get no follow-up prompt.
4. **Worker-reported blockers (task #250).** Workers call
   `swarm_report_blocker(task_number, blocked_by_task, reason)` to tell the
   IdleWatcher drone to stop nudging them on a specific task until either
   (a) the `blocked_by_task` flips to DONE, or (b) a new message lands
   in their inbox. Persisted in the `worker_blockers` SQLite table (v7
   schema migration). The watcher consults the store pre-nudge; a still-
   active blocker produces an `AUTO_NUDGE_SKIPPED` buzz entry naming the
   blocker, and the worker is not prompted. Re-reporting refreshes the
   `created_at` timestamp so the message-since window resets.

Authors of new assignment paths should go through `assign_and_start_task` (not
`task_board.assign` or the lower-level `assign_task`) unless they specifically
want queue-only semantics.

### Plan-mode gate for user-request tasks

User-channel tasks (Jira sync, email import, operator dashboard — anything
where `SwarmTask.source_worker` is empty) ship with a **plan-mode preamble**
prepended to the dispatch message by `build_task_message`
(`src/swarm/server/messages.py`). The worker is instructed to investigate
read-only, present a concrete plan via Claude Code's `ExitPlanMode` tool,
and park in `WAITING` until the operator approves from the dashboard. The
preamble explicitly tells the worker not to fire skills (`/feature`,
`/fix-and-ship`, etc.) or call `swarm_complete_task` before approval.

Worker-to-worker handoffs **bypass** the gate — `source_worker` set on the
task is the signal. That covers cross-project tasks, MCP
`swarm_create_task(target_worker=…)` calls (which tag `source_worker` via
`_handle_create_task` in `mcp/tools.py`), and the inter-worker auto-handoff
drone (`_spawn_handoff_task` in `daemon.py` was updated to tag
`source_worker=sender` so this path correctly bypasses the gate — without
that tag every auto-handoff would stall behind plan approval and defeat
the watcher's whole purpose).

Approval surface is intentionally the **existing** Claude Code plan-mode
UX (worker enters `WAITING`, operator opens the worker view, approves
in-PTY). No new approval UI was added. Dashboard already detects "plan
mode on" prompts (`server/routes/workers.py`) and the interactive Queen
already has plan-presentation handling (`queen/queen.py`).

Gated by `DroneConfig.user_request_plan_mode` (default `True`). Set to
`False` in `swarm.yaml` under `drones:` to revert to legacy fire-and-

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [miopea/swarm](https://github.com/miopea/swarm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
