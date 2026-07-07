---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a configuration repository for Claude Code. It can be consumed two ways:

1. **As a plugin** (recommended): install via `/plugin install claude-code-config` after adding the marketplace. The plugin loader sets `CLAUDE_PLUGIN_DIR` and the hook commands resolve relative to that.
2. **As a symlinked global config** (legacy path, still supported): `scripts/setup-global.sh` symlinks `agents/`, `commands/`, `skills/`, `rules/`, and `settings.json` into `~/.claude/`. Per-project use is via `scripts/setup-project.sh`.

The two modes coexist — hook command paths in `settings.json` use `${CLAUDE_PLUGIN_DIR:-<readlink fallback>}`, so they resolve in both modes without modification.

## Key Scripts

Substitute `<repo>` below with wherever you cloned this repo (commonly
`~/.config/claude-code-config/` or `~/Development/claude-code-config/`).

```bash
# Global setup (creates ~/.claude/agents, commands, skills, rules, and settings.json symlinks)
<repo>/scripts/setup-global.sh

# Project setup (run from target project directory)
<repo>/scripts/setup-project.sh <template> [template2...]
<repo>/scripts/setup-project.sh django --tooling  # + vendor the make-check tooling layer
<repo>/scripts/setup-project.sh --list       # Show templates
<repo>/scripts/setup-project.sh --check django   # Check drift + symlinks
<repo>/scripts/setup-project.sh --status     # Show current config state
<repo>/scripts/setup-project.sh --dry-run django # Preview changes

# Vendor the hard-tooling layer (Makefile, validators, git hooks, CI) into a project.
# Run by `setup-project.sh --tooling`; also works standalone.
<repo>/scripts/install-tooling.sh --hooks [--target DIR] [--dry-run]

# Merge templates (used internally by setup-project.sh)
python3 <repo>/scripts/merge-settings.py <templates-dir> base <type1> [type2...]
python3 <repo>/scripts/merge-mcp.py <mcp-templates-dir> base <type1> [type2...]
```

## Architecture

### Hooks

Hooks are configured in **two places** so the repo works in both consumption modes:

- `settings.json` (`hooks` key) — read by the symlink-global install path. Since `settings.json` is symlinked into `~/.claude/`, hooks are available in all projects.
- `hooks/hooks.json` at the repo root — read by the plugin install path (per [plugin docs](https://code.claude.com/docs/en/plugins.md)). Same shape as `settings.json`'s `hooks` object, wrapped as `{ "hooks": { ... } }`.

**Keep both files in sync** whenever you add or change a hook. Hook scripts themselves live in `scripts/hooks/` and the `${CLAUDE_PLUGIN_DIR:-$(readlink ~/.claude/settings.json | xargs dirname)}` prefix in command paths makes them resolve correctly under either mode.

#### Hook Format

Hooks use string-based matchers (e.g. `"Bash"`, `"Write|Edit"`, `"*"`). See the [official hooks reference](https://code.claude.com/docs/en/hooks.md) for the full schema and the per-event matcher fields. Repo gotcha: most events support a matcher (filtering on an event-specific field — e.g. `SessionStart` on start reason, `SessionEnd` on exit reason, `PreCompact` on `manual`/`auto`, `SubagentStop` on agent type). The events that **do not** support a matcher and must omit it are: `UserPromptSubmit`, `Stop`, `TaskCompleted`, `PostToolBatch`, `TeammateIdle`, `TaskCreated`, `WorktreeCreate`, `WorktreeRemove`, `MessageDisplay`, `CwdChanged`. Adding a `matcher` field to a no-matcher event is silently ignored per docs.

#### Available hooks

Currently configured in `settings.json`:

- **SessionStart** → `scripts/hooks/session-context.sh`: auto-loads git context (branch, recent commits, dirty files)
- **SessionEnd** → `scripts/hooks/session-end.sh`: appends session summary to `./standups/YYYY-MM-DD-log.md` for later `/standup` consumption
- **PostToolUse (Write|Edit)** → `scripts/hooks/format-on-edit.sh`: auto-formats Python files (ruff) and JS/TS files (prettier)
- **PostToolUseFailure** → `scripts/hooks/log-tool-failure.sh`: appends failed tool calls to `~/.claude/logs/tool-failures.jsonl` for later pattern analysis
- **PreToolUse (Bash)** → `scripts/hooks/dangerous-cmd-check.sh`: blocks dangerous command patterns (defense-in-depth)
- **PreCompact** → `scripts/hooks/pre-compact-state.sh`: preserves working state before context compaction
- **TaskCompleted** → `scripts/hooks/task-completed-chime.sh`: emits a terminal bell when an autonomous task completes

Available but **not configured by default** (opt-in by adding to `settings.json`):

- **UserPromptSubmit**: LLM-evaluated check that the user prompt is specific enough
- **Stop**: LLM-evaluated completeness check (tests run? linters run? TODOs left?)
- **SubagentStop**: LLM-evaluated check that a subagent completed its assigned task

The three opt-in hooks invoke an LLM on every fire and incur token cost — enable
deliberately, not by default.

CI-only utility (not a runtime hook): `scripts/hooks/check-duplicates.sh` runs from `.github/workflows/validate-config.yml` to fail CI if two agents/skills/commands share a name.

### Settings Keys


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [edjchapman/claude-code-config](https://github.com/edjchapman/claude-code-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
