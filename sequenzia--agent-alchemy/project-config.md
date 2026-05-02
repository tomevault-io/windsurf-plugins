---
trigger: always_on
description: Agent Alchemy is a monorepo that extends Claude Code into a structured development platform through markdown-as-code plugins, a real-time task dashboard, and a VS Code extension.
---

# CLAUDE.md

## Project Overview

Agent Alchemy is a monorepo that extends Claude Code into a structured development platform through markdown-as-code plugins, a real-time task dashboard, and a VS Code extension.

## Repository Structure

```
agent-alchemy/
├── claude/                    # Claude Code plugins (markdown-as-code)
│   ├── .claude-plugin/        # Plugin marketplace registry
│   ├── claude-tools/          # Claude Code Tasks & Agent Teams reference skills
│   ├── core-tools/            # Codebase analysis, deep exploration, language patterns (includes hooks/)
│   ├── dev-tools/             # Feature dev, debugging, code review, docs, changelog
│   ├── sdd-tools/             # Spec-Driven Development pipeline
│   ├── tdd-tools/             # TDD workflows: test generation, RED-GREEN-REFACTOR, coverage
│   ├── git-tools/             # Git commit automation
│   ├── plugin-tools/          # Plugin porting, adapter validation, ported plugin maintenance, ecosystem health
│   ├── opencode-tools/        # OpenCode extension creation, update, and validation toolkit
│   └── cs-tools/              # Competitive programming problem-solving and solution verification
├── agent-tools/               # Python CLI for cross-harness skill management (Typer)
├── apps/
│   └── task-manager/          # Next.js 16 real-time Kanban dashboard
├── extensions/
│   └── vscode/                # VS Code extension for plugin authoring
├── internal/docs/             # Internal documentation and analysis
└── site/                      # MkDocs documentation site (generated)
```

## Development Commands

```bash
# Task Manager
pnpm dev:task-manager          # Start dev server on port 3030
pnpm build:task-manager        # Production build

# VS Code Extension
cd extensions/vscode
npm install
npm run build                  # Build with esbuild
npm run watch                  # Watch mode
npm run package                # Package VSIX

# Agent Tools CLI
cd agent-tools
uv sync                        # Install dependencies
uv run agent-tools --help      # Run CLI
uv run pytest                  # Run tests (~295 tests)
uv run ruff check .            # Lint

# Linting
pnpm lint                      # Lint all packages
```

## Architecture Patterns

### Plugin System (claude/)

- **Skills** are defined in `SKILL.md` with YAML frontmatter and markdown body
- **Agents** are defined in `{name}.md` with YAML frontmatter (model, tools, skills)
- **Hooks** are JSON configs in `hooks/hooks.json` for lifecycle events
- Skills compose by loading other skills: `Read ${CLAUDE_PLUGIN_ROOT}/skills/{name}/SKILL.md`
- Complex skills use `references/` subdirectories for supporting materials

### Plugin Composition Patterns

- **Skill Loading**: Skills compose at runtime via `Read ${CLAUDE_PLUGIN_ROOT}/skills/{name}/SKILL.md` — prompt injection, not function calls
- **Hub-and-Spoke Teams**: `deep-analysis` spawns N explorer agents (Sonnet) + 1 synthesizer (Opus); explorers work independently, synthesizer merges with follow-ups + Bash investigation
- **Phase Workflows**: Complex skills use numbered phases with `"CRITICAL: Complete ALL N phases"` directives to prevent premature stopping
- **Reference Materials**: Large knowledge bases externalized into `references/` subdirectories (~6,000 lines total), loaded progressively when needed
- **Agent Tool Restrictions**: Architect (core-tools) and reviewer (dev-tools) agents are read-only (Glob, Grep, Read only); executor agents can write — enforces separation of concerns
- **AskUserQuestion Enforcement**: All interactive skills route user interaction through `AskUserQuestion`, never plain text output

### SDD Pipeline Patterns

- **Artifact chain**: `/create-spec` → spec markdown → `/create-tasks` → task JSON → `/execute-tasks` → code + session logs
- **Wave-based execution**: Tasks grouped by topological sort level; N agents per wave, configurable via `--max-parallel`
- **Result file protocol**: Each task-executor writes a compact `result-task-{id}.md` (~18 lines) as its last action; orchestrator polls for these instead of consuming full agent output (79% context reduction per wave)
- **Per-task context isolation**: Each agent writes to `context-task-{id}.md`; orchestrator merges into shared `execution_context.md` between waves — eliminates write contention
- **Merge mode**: `/create-tasks` uses `task_uid` composite keys for idempotent re-runs — completed tasks preserved, pending tasks updated, new tasks created
- **Session management**: Single-session invariant via `.lock` file; interrupted sessions auto-recovered with in_progress tasks reset to pending

### Session Directory Layout

```
.claude/sessions/__live_session__/       # Active execution session
├── execution_plan.md                    # Wave plan from orchestrator
├── execution_context.md                 # Shared learnings across tasks
├── task_log.md                          # Per-task status, duration, tokens
├── progress.md                          # Real-time progress tracking
├── tasks/                               # Archived completed task JSONs
├── context-task-{id}.md                 # Per-task context (ephemeral)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sequenzia/agent-alchemy](https://github.com/sequenzia/agent-alchemy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
