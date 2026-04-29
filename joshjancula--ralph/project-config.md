---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code), Cursor (cursor-agent), Codex and any AI agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code), Cursor (cursor-agent), Codex and any AI agents when working with code in this repository.

## Overview

Ralph is a framework for organizing AI coding assistant workflows. It provides:
- **Plan-first loop:** Markdown todo lists executed by AI assistants (Cursor, Claude Code, or Codex)
- **Orchestration:** Multi-stage pipelines (research → design → implementation → review) with artifact handoffs
- **Agents:** Prebuilt agent profiles for specialized work (research, architect, implementation, code-review, qa, security)
- **Dashboard:** Optional Node UI for monitoring plan execution and artifact generation

Ralph is installed into projects via `./install.sh`. The installer copies shared scripts to `.ralph/`, runtime-specific runners to `.cursor/ralph`, `.claude/ralph`, `.codex/ralph`, `.opencode/ralph`, and agents/rules/skills to `.cursor/agents`, `.claude/agents`, `.opencode/agents`, etc.

> **Symlink note (this repo only):** `.cursor`, `.claude`, `.codex`, and `.opencode` at the repo root are symlinks to `bundle/.cursor`, `bundle/.claude`, `bundle/.codex`, and `bundle/.opencode`. Editing files under `bundle/.cursor/` (or `bundle/.claude/`, `bundle/.codex/`, `bundle/.opencode/`) automatically updates the symlinked paths — no manual copy or sync is needed. Similarly, `.ralph/` files are hardlinked to `bundle/.ralph/`, so edits there are also immediately reflected.

## Key Commands

### Run tests (Bats shell test suite)

```bash
# Full test suite
bash scripts/setup-test-fixtures.sh
bats tests/bats/*.bats

# Run a specific test file
bats tests/bats/run-plan-invoke.bats

# Set required env var for all runs (suppresses usage risk prompt)
RALPH_USAGE_RISKS_ACKNOWLEDGED=1 bats tests/bats/run-plan-unified.bats
```

The test suite uses Bats (Bash Automated Testing System). Tests live in `tests/bats/` and cover:
- Installation and setup (`install.bats`, `install-lib.bats`)
- Plan execution (`run-plan-*.bats`)
- Orchestration (`orchestration-*.bats`)
- Human interaction (`human-interaction.bats`)
- MCP server (`mcp-server.bats`)
- Agent scaffolding (`new-agent.bats`, `bundle-new-agent-scripts.bats`)

### Run the installer

```bash
# Full install (all runtimes + dashboard)
./install.sh

# Install to a specific target repo
./install.sh /path/to/repo

# Claude and shared scripts only (no Cursor/Codex)
./install.sh --claude --shared

# Dry-run (print what would be copied)
./install.sh -n
```

Submodule, subtree, partial installs, and cleanup: [docs/INSTALL.md](docs/INSTALL.md).

### Run a plan (`run-plan.sh`)

Invoke **`.ralph/run-plan.sh`** with **`--plan`** (required). Pass **`--runtime`** unless **`RALPH_PLAN_RUNTIME`** is set or you rely on the interactive runtime prompt (TTY). Pass **`--workspace <path>`** for an explicit repo root; if omitted, the workspace defaults to the current working directory. The parser in `bundle/.ralph/bash-lib/run-plan-args.sh` rejects unknown arguments and does not accept positional workspace or plan paths. See [README.md](README.md) for typical commands and canonical examples.

### Ralph Dashboard (Node UI)

In this repository, develop and test from **`ralph-dashboard/`** at the repo root:

```bash
cd ralph-dashboard
npm install
npm run build
npm start
```

Use **`PORT=8124 npm start`** instead of **`npm start`** when you need a different port.

After **`install.sh`** copies Ralph into another project, the dashboard lives at **`.ralph/ralph-dashboard/`**. From that project root run **`cd .ralph/ralph-dashboard && npm install`**, then **`npm run build`** and **`npm start`** (use **`PORT=8124 npm start`** to override the port).

The dashboard reads plan state, logs, and artifacts from `.ralph-workspace/` and provides a UI for monitoring orchestration runs.

### Validate orchestration schema

```bash
bash scripts/validate-orchestration-schema.sh <orchestration-file.orch.json>
```

## Recommended optional tools

- `fzf` - Install for arrow-key menus in interactive prompts (brew install fzf / apt install fzf). Set RALPH_SKIP_FZF_HINT=1 to silence the install hint.
- `python3` - Required for CLI session resume functionality (captures session IDs from tool output).

## Architecture

### Bundle structure

```
bundle/
  .ralph/              # Shared across all runtimes
    run-plan.sh       # Unified plan executor (required by all)
    orchestrator.sh   # Multi-stage orchestration runner
    orchestration-wizard.sh
    cleanup-plan.sh
    new-agent.sh
    bash-lib/         # Helpers: plan-todo, run-plan-env, run-plan-invoke-*.sh, install-ops.sh, etc.
    mcp-server.sh     # Bash MCP server
    agent-config-tool.sh
  .cursor/
    ralph/            # Cursor-specific runner thin wrapper
    agents/           # Cursor agent profiles
      research/
      architect/
      implementation/
      code-review/
      qa/
      security/
  .claude/
    ralph/            # Claude Code-specific runner thin wrapper
    agents/           # Claude agent profiles (same 6 agents)
    rules/
      no-emoji.md
    skills/
      repo-context/SKILL.md
  .codex/
    ralph/            # Codex-specific runner thin wrapper

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JoshJancula/ralph](https://github.com/JoshJancula/ralph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
