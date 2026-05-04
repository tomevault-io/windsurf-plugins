---
trigger: always_on
description: A live terminal dashboard for monitoring all your Claude Code sessions at a glance. Like `htop`, but for Claude Code.
---

# cctop, Claude Code Sessions Dashboard

A live terminal dashboard for monitoring all your Claude Code sessions at a glance. Like `htop`, but for Claude Code.

## Why

Power users run multiple Claude Code sessions simultaneously, one refactoring a module, another writing tests, a third researching an API. You end up tab-switching between terminals just to check "is it done yet?" or "is it stuck waiting for me?" There's no central place to see what's happening across sessions.

## What It Does

Installs a lightweight hook into Claude Code that tracks session activity in real time. A companion TUI dashboard (`cctop`) displays all active sessions in a single live-updating table:

- **Status**, see at a glance whether each session is idle (waiting for you), thinking, editing files, running commands, searching the web, or spawning subagents
- **Project & branch**, know which codebase and branch each session is working in
- **Context usage**, monitor how much of the context window has been consumed, so you can wrap up or start fresh before hitting limits
- **Tool count**, track how many tool calls a session has made
- **Model**, which Claude model each session is using
- **Last messages**, peek at the most recent user prompt and Claude response without switching terminals

Sessions that go quiet for 1+ hour are marked stale. Sessions that end clean up after themselves automatically.

## Who It's For

Anyone running more than one Claude Code session at a time, or anyone who wants a quick overview of what's happening without context-switching into each terminal.

## Project Structure

- `plugin/`, distribution files (only this directory gets installed)
  - `plugin/scripts/cctop-hook.sh`, hook handler, writes `~/.cctop/<session-id>.json`
  - `plugin/scripts/cctop_dashboard.py`, Textual TUI app (run with `uv run --script`)
  - `plugin/scripts/cctop-poller.py`, background transcript poller
  - `plugin/scripts/launch-cctop.sh`, convenience launcher
  - `plugin/hooks/hooks.json`, registers the hook for 7 events
  - `plugin/.claude-plugin/plugin.json`, plugin manifest
- `.claude-plugin/marketplace.json`, local marketplace manifest (points to `./plugin/`)
- `tests/test_cctop_dashboard.py`, TUI tests
- `install.sh`, reinstalls plugin into Claude's cache
- `plans/`, **gitignored**, PRDs and design docs (never commit these, never `git add` them)
- `BACKLOG.md`, numbered feature backlog with completion tracking

## Reference Docs

The `reference/` directory contains Claude Code internals documentation, split by topic. **Read these on-demand**, don't load them all upfront, just read the one relevant to your current task:

| File | When to read |
|---|---|
| `reference/hooks-api.md` | Writing or debugging hooks, events, stdin fields, output format |
| `reference/transcript-format.md` | Parsing JSONL transcripts, entry types, field shapes, path encoding |
| `reference/sessions-index.md` | Reading the sessions index, schema, customTitle timing |
| `reference/plugin-system.md` | Plugin install/dev workflow, manifests, cache, gotchas |
| `reference/session-data-files.md` | Tool counts and session-status JSON files |
| `reference/slash-commands.md` | Slash command categories, transcript formats (3 variants), hook invisibility, poller parsing pipeline |

## Textual TUI Development

When writing or modifying Textual-based code (the dashboard, widgets, tests), always use the `textual-tui-dev` skill first to load best practices for architecture, workers, reactive state, testing patterns, etc.

## Installing After Changes

The plugin runs from a **copy** in `~/.claude/plugins/cache/`, not from this directory.
After editing any file under `plugin/`, changes must be installed before testing.

The user runs dev install from their own terminal:
```bash
./install.sh --dev --wt <prefix>   # from main repo, auto-finds worktree by prefix
./install.sh --dev                  # from inside a worktree directly
```

**Do NOT run `./install.sh --dev` yourself.** The user handles installation from their iTerm2 terminal. After making changes to plugin files, tell the user the changes are ready for DI (dev install). "DI" is shorthand for dev install.

## Releasing

Use `release.sh` for version bumps and tagging. The script handles the mechanical parts, you write the changelog.

1. `./release.sh bump <version>` — updates `plugin.json`, prints git log since last tag
2. Read the git log output and write a human-readable `CHANGELOG.md` entry. Prepend the new entry to the file. Use format: `## vX.Y.Z — YYYY-MM-DD`
   - Group features into thematic subgroups with punchy headers (e.g. "Status Detection, Know What Every Session Is Actually Doing")
   - Focus on what changed for the user, not implementation details (no config file paths, hook event names, or keybindings)
   - No self-praise or hype ("major upgrade", "significant improvement"), just describe what's new
   - Cross-reference `plans/pr-groups.md` and `BACKLOG.md` to make sure nothing is missed
3. `git add plugin/.claude-plugin/plugin.json CHANGELOG.md`
4. `./release.sh tag` — commits, tags, pushes

## Writing Style

- Never use emdashes (—) in any language. Use commas in prose, regular dashes (-) in lists.

### Hebrew Announcements


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DeanLa/cctop](https://github.com/DeanLa/cctop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
