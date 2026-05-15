---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is pokegents

pokegents is a multi-session Claude Code launcher and session manager. It wraps the `claude` CLI with profile-based configuration, terminal theming (iTerm2 on macOS), session tracking, and Claude Code hook integration. Users run multiple concurrent Claude Code sessions across different projects (e.g. 3 client sessions + 1 personal session simultaneously).

## Architecture

**Entry point:** installed `pokegents`/`pokegent` shims set `POKEGENTS_ROOT`, source `pokegent.sh`, and dispatch to the `pokegent()` function. Users should not need to edit or source shell rc files.

**Code lives in the repo, user data lives in `~/.pokegents/`:**
- `~/.pokegents/profiles/*.json` — per-profile config (cwd, system_prompt, emoji, color, iterm2_profile, add_dirs)
- `~/.pokegents/running/*.json` — active session registry (profile, session_id, pid, tty, display_name)
- `~/.pokegents/status/*.json` — structured state from hooks (state, detail, cwd, timestamp, last_summary)
- `~/.pokegents/history/*.json` — last 5 sessions per profile with timestamps and first-message summaries

**Hooks (in `hooks/`):**
- `status-update.sh` — registered on all Claude Code lifecycle events (PreToolUse, PostToolUse, Stop, SessionStart, SessionEnd, etc.). Writes structured JSON to `~/.pokegents/status/<session_id>.json` with state (running/idle/error/permission/waiting).
- `statusline.sh` — renders profile emoji + title in profile RGB color for Claude's status bar.

**Hook paths are set in `~/.claude/settings.json`** and point to this repo's `hooks/` directory. The `install.sh` script wires this up.

**Session lifecycle:** launch → register in running/ → set terminal tab color/title (iTerm2) → run claude → on exit: cleanup running file, save to history, restore terminal profile.

**Duplicate handling:** if a profile is already running, pokegent prompts to rename existing sessions and name the new one (supporting patterns like 3 concurrent "client" sessions named "pinecone", "int chroma", "int pine").

## Build / Install / Test

```bash
# Install (creates data dirs and CLI shims; no shell rc mutation)
./install.sh

# Developer/source build
POKEGENTS_DEV_BUILD=1 ./install.sh
```

## Roadmap and Goals

The owner wants pokegents to evolve into a full agent orchestration platform. Priority areas:

1. **Hooks integration** — the status-update hook captures events but nothing reads/reacts to the data yet. Goal: richer hook logic (notifications, auto-actions, cross-session triggers).

2. **Dashboard UI** — a viewer for all active agents and their progress. TBD whether terminal TUI or web. The data is already there in `running/` + `status/`. Existing ecosystem references: claude-code-monitor (TUI + mobile web), claude-cockpit (VS Code sidebar), claude-code-ui (web dashboard).

3. **Advanced profile customization** — sub-project support within profiles, profile inheritance, composable config layers (e.g. "client profile + pinecone sub-project overrides").

4. **Git worktree + cross-agent communication** — agents on different worktrees of the same repo sharing context and coordinating. The `-w` flag exists but just passes through to claude. Key patterns from the ecosystem:
   - Claude Code's experimental Agent Teams feature (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`) uses file-based mailbox inboxes with append-and-poll messaging under `~/.claude/teams/`
   - HCOM uses hook-based messaging: `agent → hooks → db → hooks → other agent` with direct messages, broadcasts, @-mentions, collision detection
   - agent-collab-mcp uses an MCP server as coordination layer so agents communicate via tool calls
   - Community consensus: 3-5 concurrent agents is the sweet spot, decomposition quality matters more than agent count

5. **Broader extensibility** — plugin system, MCP server integration, multi-machine coordination (currently an open gap in the ecosystem).

## Key Conventions

- `--dangerously-skip-permissions` is configurable per-profile (`skip_permissions` field) and globally in `~/.pokegents/config.json`
- `POKEGENTS_PROFILE_NAME` env var is set on launch so hooks/statusline can read the active profile
- `POKEGENTS_ROOT` and `POKEGENTS_DATA` env vars are passed through to claude so hooks can reference both repo code and user data
- iTerm2 restore profile is configurable in `~/.pokegents/config.json` (`iterm2_restore_profile`)
- Session IDs are lowercase UUIDs from `uuidgen`

## Deep Architecture Reference

### Hook System (`hooks/status-update.sh`)

**State machine:** `idle → busy → done → needs_input → error`

| Transition | Trigger |
|-----------|---------|
| `* → busy` | UserPromptSubmit, PreToolUse, PostToolUse |
| `busy → done` | Stop, Notification(idle_prompt) when current=busy |
| `* → needs_input` | PermissionRequest only (never from idle_prompt) |
| `* → error` | StopFailure |
| `* → idle` | SessionStart (unless previous state was busy/compacting) |

**Critical rule:** `set -e` is NOT used. Every jq call has `2>/dev/null || fallback`. A crashing hook blocks ALL Claude operations — we learned this from a 7-hour error loop.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tRidha/pokegents](https://github.com/tRidha/pokegents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
