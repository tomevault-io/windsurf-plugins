---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

dotbot is a structured AI-assisted development framework built entirely in **PowerShell 7+**. It wraps AI coding workflows in managed, auditable processes with two-phase execution (analysis → implementation), per-task git worktree isolation, and a web dashboard for monitoring.

## Commands

**Always use `pwsh` (PowerShell 7), never `powershell` (5.1).** PS 5.1 cannot handle UTF-8 files without BOM.

```bash
# Install/update dotbot globally (from repo root)
pwsh install.ps1

# Run tests (layers 1-3, no Claude credentials needed)
pwsh tests/Run-Tests.ps1

# Run a specific test layer
pwsh tests/Run-Tests.ps1 -Layer 1       # Structure tests
pwsh tests/Run-Tests.ps1 -Layer 2       # Component tests
pwsh tests/Run-Tests.ps1 -Layer 3       # Mock Claude tests
pwsh tests/Run-Tests.ps1 -Layer 4       # E2E (requires ANTHROPIC_API_KEY)

# Initialize dotbot in a project
dotbot init
dotbot init --profile dotnet

# Launch the web UI (default port 8686, auto-selects if busy)
.bot\go.ps1
.bot\go.ps1 -Port 9000   # Use a specific port
```

## Architecture

Framework code lives in `core/` at the repo root; workflow content lives under `workflows/<name>/`. Both get copied to `.bot/` on `dotbot init` in target projects. The `.bot/` directory in this repo is gitignored — **never edit files in `.bot/`**, always edit the source in `core/` (framework) or `workflows/<name>/` (workflow content).

### Three Core Systems (`core/`)

**MCP Server** (`core/mcp/`) — Pure PowerShell MCP server (stdio transport, protocol 2024-11-05). Tools are auto-discovered from `tools/{tool-name}/` subdirectories, each containing `metadata.yaml` + `script.ps1`. 26 tools for task, session, plan, and dev management.

**Web UI** (`core/ui/`) — Pure PowerShell HTTP server with vanilla JS frontend. Dashboard tabs: Overview, Product, Workflow, Processes, Settings, Roadmap. Default port 8686 (auto-selects next available if busy).

**Runtime** (`core/runtime/`) — Manages Claude CLI invocations as tracked processes. `launch-process.ps1` is the unified entry point with process types: `task-runner`, `planning`, `commit`, `task-creation`. Includes `WorktreeManager.psm1` for git worktree isolation and `ClaudeCLI.psm1` / `ProviderCLI.psm1` for provider wrappers.

### Recipes & Agents (`core/`)

- **Agents** (`core/agents/`): `implementer/`, `planner/`, `reviewer/`, `tester/` — TDD-focused AI personas
- **Skills** (`core/skills/`): Reusable technical guidance (e.g., `write-unit-tests/`)
- **Framework prompts** (`core/prompts/`): `98-analyse-task.md` (pre-flight analysis) and `99-autonomous-task.md` (execution) are the core two-phase workflow; `90-commit-and-push.md`, `91-new-tasks.md`, `92-steering-protocol.include.md`, `05-retrospective-task.md` round out the framework set.
- **Workflow prompts** live under each `workflows/<name>/recipes/prompts/` and stay workflow-scoped.

### Two-Phase Execution

1. **Analysis** (`98-analyse-task.md`): Explores codebase, identifies affected files, builds context package, may propose task splits or request user input. Task moves: `todo → analysing → analysed`.
2. **Implementation** (`99-autonomous-task.md`): Consumes pre-built context, writes code, runs tests, commits with `[task:XXXXXXXX]` tag. Task moves: `analysed → in-progress → done`.

### Git Worktree Isolation

Each task gets its own branch (`task/{short-id}-{slug}`) and worktree (`../worktrees/{repo}/task-{short-id}-{slug}/`). On completion, the task branch is squash-merged to main and the worktree is cleaned up.

### Hooks

- `core/hooks/verify/` — Numbered verification scripts (framework-shared): `00-privacy-scan.ps1` (gitleaks), `01-git-clean.ps1`, `02-git-pushed.ps1`, `03-check-md-refs.ps1`, `04-framework-integrity.ps1`
- `core/hooks/dev/` — `Start-Dev.ps1`, `Stop-Dev.ps1` for dev environment lifecycle
- `core/hooks/scripts/` — `commit-bot-state.ps1`, `steering.ps1`

## Adding MCP Tools

1. Create folder: `core/mcp/tools/your-tool-name/`
2. Add `metadata.yaml` (snake_case name, JSON Schema), `script.ps1` (PascalCase `Invoke-YourToolName` function), and `test.ps1`
3. Server auto-discovers the tool — no registration needed

Naming: folder=`kebab-case`, YAML name=`snake_case`, function=`Invoke-PascalCase`.

## Test Pyramid

| Layer | File | What it tests | Credentials |
|-------|------|---------------|-------------|
| 1 | `Test-Structure.ps1` | Dependencies, installation, platform functions | None |
| 2 | `Test-Components.ps1` | MCP tools, UI APIs, file structure | None |
| 3 | `Test-MockClaude.ps1` | Analysis/execution flows with mock Claude CLI | None |
| 4 | `Test-E2E-Claude.ps1` | Full end-to-end with real Claude API | `ANTHROPIC_API_KEY` |

CI runs layers 1-3 on push/PR across Windows, macOS, Linux. Layer 4 runs on schedule or manual trigger.

## Dev Cycle

After every set of changes, run the install script from the project root so the user can test against the latest build, then run level 1-3 tests:

```bash
# 1. Install/update (from dotbot repo root)
pwsh install.ps1

# 2. Run tests (layers 1-3)
pwsh tests/Run-Tests.ps1
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andresharpe/dotbot](https://github.com/andresharpe/dotbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
