---
trigger: always_on
description: This file provides guidance to AI coding assistants when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to AI coding assistants when working with code in this repository.

## Project Overview

Parsidion is the source repository for an agent-agnostic markdown knowledge vault: skills, agents, hook scripts, search/index tools, and visualizer/MCP integrations that give coding agents persistent memory. Claude Code remains the primary installed adapter today, but the core vault tooling is runtime-agnostic.

## Installed vs Source Paths

| Component | Source (this repo) | Installed to |
|---|---|---|
| Installer | `install.py` | run in-place (`uv run install.py`) |
| Parsidion vault skill | `skills/parsidion/` | `~/.claude/skills/parsidion/` |
| Research agent | `agents/research-agent.md` | `~/.claude/agents/` |
| Hook scripts | `skills/parsidion/scripts/` | referenced from `~/.claude/settings.json` |
| Vault | (generated) | `~/ClaudeVault/` (or custom path) |

Use `install.py` to sync changes from this repo to the installed locations. After editing source files, run:

```bash
uv run install.py --force --yes
```

## Running Scripts

All scripts use Python stdlib only (no third-party dependencies). Run them with `uv`:

```bash
# Install (or reinstall after source changes)
uv run install.py                    # interactive
uv run install.py --force --yes      # non-interactive reinstall
uv run install.py --dry-run          # preview only
uv run install.py --uninstall        # remove skill, agent, hooks, and launchd plist / cron job
uv run install.py --uninstall-hooks  # remove only managed hook registrations from settings.json

# Schedule nightly auto-summarization (launchd on macOS, cron on Linux)
uv run install.py --schedule-summarizer
uv run install.py --schedule-summarizer --summarizer-hour 3  # run at 3 AM

# Also rebuild visualizer graph.json each night
uv run install.py --schedule-summarizer --rebuild-graph
uv run install.py --schedule-summarizer --rebuild-graph --graph-include-daily

# Install vault CLIs as global commands — cross-platform via uv tool
uv run install.py --install-tools    # runs uv tool install --editable ".[tools]"
# OR manually from the repo root:
uv tool install --editable ".[tools]"

# Rebuild the vault index (after creating/renaming/deleting notes)
uv run --no-project ~/.claude/skills/parsidion/scripts/update_index.py

# Rebuild index AND regenerate visualizer graph.json in one pass
uv run --no-project ~/.claude/skills/parsidion/scripts/update_index.py --rebuild-graph

# Also include Daily notes in the graph
uv run --no-project ~/.claude/skills/parsidion/scripts/update_index.py --rebuild-graph --graph-include-daily

# Summarize queued sessions (from a terminal outside Claude Code)
uv run --no-project ~/.claude/skills/parsidion/scripts/summarize_sessions.py
uv run --no-project ~/.claude/skills/parsidion/scripts/summarize_sessions.py --dry-run

# Summarize from inside a Claude Code session (unset CLAUDECODE to allow nesting)
env -u CLAUDECODE uv run --no-project ~/.claude/skills/parsidion/scripts/summarize_sessions.py

# Search vault notes (after uv tool install --editable ".[tools]")
vault-search "hook patterns" -n 5            # semantic, top 5
vault-search -n 5 -r "hook patterns"         # semantic, rich output
vault-search -f Patterns                     # metadata: by folder
vault-search -T python -d 7                  # metadata: by tag + recency
vault-search --folder Patterns --tag python  # metadata: long form still works
vault-search --grep "dedup_threshold"        # full-text body search (case-insensitive)
vault-search --grep "FLOCK" --grep-case      # full-text body search (case-sensitive)
vault-search --interactive                   # interactive curses TUI
VAULT_SEARCH_FORMAT=rich VAULT_SEARCH_MIN_SCORE=0.5 vault-search "query"  # env vars

# Scaffold a new vault note (after uv tool install --editable ".[tools]")
vault-new --type pattern --title "My Pattern" --project myproj --tags python,vault --open
vault-new --type debugging --title "Fix X Error" --tags sqlite

# Vault analytics (after uv tool install --editable ".[tools]")
vault-stats --summary              # note counts, growth, top tags
vault-stats --stale                # notes with no incoming links older than 30 days
vault-stats --top-linked           # most-referenced notes
vault-stats --by-project           # note counts per project
vault-stats --growth               # notes added per week
vault-stats --tags                 # tag frequency cloud
vault-stats --pending              # pending queue status (count, sources, oldest entry)
vault-stats --graph                # knowledge graph metrics (avg degree, hubs, orphans)
vault-stats --hooks 50             # last 50 hook events from hook_events.log
vault-stats --weekly               # generate weekly rollup note from daily notes
vault-stats --monthly              # generate monthly rollup note from daily notes
vault-stats --timeline 90          # activity bar chart for last 90 days
vault-stats --summarizer-progress  # live feedback from running summarize_sessions.py
vault-stats --dashboard            # full combined dashboard

# Review pending sessions before summarization (after uv tool install --editable ".[tools]")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paulrobello/parsidion](https://github.com/paulrobello/parsidion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
