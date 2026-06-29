---
trigger: always_on
description: This project uses OpenWolf for context management. Read and follow .wolf/OPENWOLF.md every session. Check .wolf/cerebrum.md before generating code. Check .wolf/anatomy.md before reading files.
---

# OpenWolf

@.wolf/OPENWOLF.md

This project uses OpenWolf for context management. Read and follow .wolf/OPENWOLF.md every session. Check .wolf/cerebrum.md before generating code. Check .wolf/anatomy.md before reading files.


# OTGW-firmware: Claude Instructions

---

## Task Management (MANDATORY)

**Every piece of work must have a backlog task before any code is written. No exceptions.**

**Always use the `backlog` CLI for task operations on this project. Do NOT use the `mcp__backlog__*` MCP server.** The MCP server indexes only one worktree at a time and serves stale cached state across worktrees (verified 2026-05-05: MCP returned a pre-edit "In Progress" snapshot of TASK-514 long after the CLI marked it Done on disk in another tree). The "Backlog.md: always use the CLI" section near the bottom of this file is the canonical statement; this paragraph is a reminder so the rule is the first thing seen.

```bash
# Before writing any code:
backlog search "<topic>" --plain           # 1. Find existing task
backlog task create "Title" -d "..." --ac "..."  # 2. Create if none
backlog task edit <id> -s "In Progress" -a @claude  # 3. Start it
backlog task edit <id> --plan "..."        # 4. Write plan, share with user, WAIT for approval

# During implementation:
backlog task edit <id> --check-ac 1        # Mark ACs done as you go
backlog task edit <id> --append-notes "..."  # Log progress

# When done:
backlog task edit <id> --final-summary "..." # PR description
backlog task edit <id> -s Done
```

**CRITICAL: NEVER edit task files in `backlog/tasks/` directly. Always use the `backlog` CLI.**

Two hooks enforce this contract — they fail closed, you don't have to remember:

- `.claude/hooks/backlog-mcp-guard.py` — PreToolUse guard wired in `.claude/settings.json`. Blocks `Edit/Write/MultiEdit` on `backlog/tasks/*.md` (direct file edits are never allowed). Does NOT block `backlog` CLI invocations — CLI is the preferred interface.
- `.githooks/commit-msg` — git hook that fails the commit if its message references `TASK-NNN` without a matching `backlog/tasks/task-NNN*.md` file in the index. Catches the failure mode where a code commit lands but its task record stays untracked. Install once per clone with `git config core.hooksPath .githooks`. Bypass with `git commit --no-verify` for emergencies (document why in the message).

For the full CLI reference (all commands, AC management, DoD, multi-line input): read `docs/guides/backlog-cli.md`.

Before marking a task `Done`, run through `docs/guides/pr-checklist.md`. "Builds clean" is the lowest bar; the checklist captures the hardware / browser / MQTT smoke tests that build-clean doesn't see.

**Known bug:** `backlog task list` returns empty. Use `backlog task <id> --plain` or read `backlog/tasks/` directly.

## Task pickup (MANDATORY)

**When picking up any task from the backlog — whether newly created or already existing — the first action before any code, research, or file reading is:**

```bash
backlog task edit <id> -s "In Progress" -a @claude
```

This makes the task visible in the correct board column immediately. Skipping this step leaves the task in "To Do" while it is actually being worked on, which creates false visibility for the user and breaks board accuracy.

## Auto-advance to next task (project policy)

After completing a task (or reaching a blocking state with no self-verifiable ACs remaining), **immediately analyse the backlog and pick up the highest-priority actionable task** without waiting for the user to prompt. Apply the following selection order:

1. Highest-priority task with all non-field-validation ACs unblocked (can be started and verified without hardware).
2. If all open tasks are blocked on field validation or external input, report that state and idle.

Exceptions — do NOT auto-advance:
- User has explicitly asked you to stop or wait.
- The next task requires a plan approval checkpoint (per KISS principle: share choices so user decides on complexity).
- The next task is a cross-worktree master-plan task (requires the one-plan-then-two-agents protocol).

---

## MCP Servers

Two MCP servers are wired into Claude Code for this project:

- **`mcp__backlog__*`** — task management. Started locally by Claude Code itself (stdio).
- **`mcp__discord-mcp__*`** — Discord I/O for `#dev-sat-mqtt` and the support channels. Runs as a long-lived Docker container (`saseq/discord-mcp`) on `http://localhost:8085/mcp`, pre-loaded with `DISCORD_TOKEN` from the Windows user environment. Most-used tools: `read_messages`, `send_message`, `get_server_info`, `read_private_messages`, `send_private_message`. There is no separate login tool — the container handshakes on its own at startup. For attachment contents (logs, screenshots), see `.claude/commands/backlog_discord.md` (Phase 1b) and `.claude/commands/check_otgw_issues.md` (Phase 1d).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rvdbreemen/OTGW-firmware](https://github.com/rvdbreemen/OTGW-firmware) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
