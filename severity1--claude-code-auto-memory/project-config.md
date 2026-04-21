---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

<!-- AUTO-MANAGED: project-description -->
## Overview

**claude-code-auto-memory** - A Claude Code plugin that automatically maintains CLAUDE.md files (or AGENTS.md for multi-agent repos) as codebases evolve. Tracks file changes via hooks, spawns agents to update memory, and provides skills for codebase analysis.

Key features:
- Real-time file tracking via PostToolUse hooks
- Stop hook integration to trigger memory updates
- Codebase analyzer skill for initial setup
- Memory processor skill for ongoing updates
- AGENTS.md support for multi-agent workflows (OpenAI Codex, Gemini, etc.)

<!-- END AUTO-MANAGED -->

<!-- AUTO-MANAGED: build-commands -->
## Build & Development Commands

```bash
# Install dependencies
uv sync

# Run tests
uv run pytest tests/ -v

# Run single test file
uv run pytest tests/test_hooks.py -v

# Lint code
uv run ruff check .

# Format code
uv run ruff format .

# Type check
uv run mypy scripts/
```

<!-- END AUTO-MANAGED -->

<!-- AUTO-MANAGED: architecture -->
## Architecture

```
.claude-plugin/          # Plugin manifest
  plugin.json            # Plugin metadata and version
agents/
  memory-updater.md      # Agent that updates CLAUDE.md files
commands/
  init.md                # /auto-memory:init command
  calibrate.md           # /auto-memory:calibrate command
  status.md              # /auto-memory:status command
  sync.md                # /auto-memory:sync command
hooks/
  hooks.json             # Hook registration
scripts/
  post-tool-use.py       # Tracks file edits to dirty-files
  trigger.py             # Consolidated handler for PreToolUse, Stop, and SubagentStop
skills/
  codebase-analyzer/     # Initial CLAUDE.md generation
  memory-processor/      # Ongoing CLAUDE.md updates
  shared/                # Shared references
tests/
  test_hooks.py          # PostToolUse hook behavior tests
  test_trigger.py        # trigger.py unit tests
  test_integration.py    # Plugin structure tests
  test_skills.py         # Skill validation tests
```

Data flow:
1. User edits files via Edit/Write tools or git operations (rm, mv)
2. PostToolUse hook appends paths to session-specific `.claude/auto-memory/dirty-files-{session_id}` (falls back to `dirty-files` without session_id)
3. PreToolUse hook (gitmode only) denies git commit until dirty files processed
4. Stop hook detects dirty files, blocks Claude, requests agent spawn
5. memory-updater agent processes files and updates CLAUDE.md
6. SubagentStop hook auto-commits CLAUDE.md (if configured), clears dirty-files, cleans up stale session files

Configuration:
- Trigger modes: `default` (after every turn) or `gitmode` (only after git commits)
- `memoryFiles`: Array specifying which memory file(s) to maintain - `["CLAUDE.md"]` (default), `["AGENTS.md"]`, or `["CLAUDE.md", "AGENTS.md"]` (AGENTS.md holds content, CLAUDE.md redirects)
- `autoCommit`: When true, auto-commits memory file changes after memory-updater completes
- `autoPush`: When true (requires autoCommit), pushes commits to remote
- Config stored in `.claude/auto-memory/config.json`
- Init wizard interactively configures triggerMode, memoryFiles, autoCommit, and autoPush, then updates `.gitignore` to exclude `dirty-files*` tracking files

<!-- END AUTO-MANAGED -->

<!-- AUTO-MANAGED: conventions -->
## Code Conventions

- **Python**: 3.9+ with type hints, snake_case naming
- **Imports**: Group stdlib, then third-party, then local
- **Docstrings**: Module-level docstrings explain purpose
- **Hooks**: Zero output for PostToolUse/SubagentStop (token cost), JSON output for Stop/PreToolUse
- **Hook routing**: Use hook_event_name from stdin JSON to differentiate behavior
- **Hook commands**: Use python3 with fallback (`python3 script.py || python script.py`) for cross-platform compatibility
- **Skills/Commands**: YAML frontmatter with name/description
- **Line length**: 100 characters (ruff config)
- **Testing**: pytest with descriptive test names (test_verb_condition)

<!-- END AUTO-MANAGED -->

<!-- AUTO-MANAGED: patterns -->
## Detected Patterns

- **Hook Consolidation Pattern**: Single trigger.py handles PreToolUse, Stop, and SubagentStop hooks, routing based on hook_event_name
- **Initialization Guard Pattern**: `plugin_initialized()` in post-tool-use.py, handle_stop(), and handle_pre_tool_use() gates all activity on config.json existence — projects that never ran /auto-memory:init are fully inert
- **Hook Lifecycle Pattern**: PostToolUse tracks → Stop/PreToolUse blocks → Agent spawns → SubagentStop cleans up (cleanup gated on dirty-files only, not config.json, to prevent infinite loops)
- **Separation of Concerns**: PostToolUse (silent tracking) vs Stop/PreToolUse (blocking with output) vs SubagentStop (cleanup)
- **Dirty File Pattern**: Dict-keyed deduplication (path as key), batch processing at turn end, session-isolated via `dirty-files-{session_id}`; commit context entries overwrite plain path entries when present
- **Skill Pattern**: YAML frontmatter + markdown body with algorithm sections
- **Template Pattern**: AUTO-MANAGED markers for updatable sections

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [severity1/claude-code-auto-memory](https://github.com/severity1/claude-code-auto-memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
