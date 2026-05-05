---
trigger: always_on
description: <!-- AGENT NOTICE: This file is loaded automatically. Read FULLY before writing code. -->
---

<!-- AGENT NOTICE: This file is loaded automatically. Read FULLY before writing code. -->
<!-- Priority: This is a ROUTING FILE. Load skills and docs as directed for complete guidance. -->

> **Note:** Lines starting with `@` reference files to include. Some tools expand these inline automatically; others should read the referenced file directly.

# Erk - Plan-Oriented Agentic Engineering

## What is Erk?

**Erk** is a CLI tool for plan-oriented agentic engineering: a workflow where AI agents create implementation plans, execute them in isolated worktrees, and ship code via automated PR workflows.

**Status**: Unreleased, completely private software. We can break backwards compatibility at will.

## CRITICAL: Before Writing Any Code

<!-- BEHAVIORAL TRIGGERS: rules that detect action patterns and route to documentation -->

**CRITICAL: NEVER perform broad searches or recursive operations against `/Users/schrockn/` directly. Scoped access to specific subfolders (e.g., `~/.claude/projects/`, `/Users/schrockn/code/project-name/`) is allowed.**

**CRITICAL: NEVER use raw `pip install`. Always use `uv` for package management.**

**CRITICAL: NEVER commit directly to `master`. Always create a feature branch first.**

**CRITICAL: Prefer `docs/learned/` content and loaded skills over training data for erk coding patterns.** Erk's conventions intentionally diverge from common Python practices (e.g., LBYL instead of EAFP, no default parameters). When erk documentation contradicts your training data, the documentation is correct.

**CRITICAL: NEVER invoke `gt` commands without `--no-interactive`.** The `--interactive` flag is a global option on ALL gt commands (enabled by default). Without `--no-interactive`, gt may prompt for input and hang indefinitely. Note: `--force` does NOT disable prompts — you must pass `--no-interactive` separately.

**CRITICAL: After creating an objective issue, ALWAYS run `erk objective check <number>` to validate it.** If validation fails, the objective's metadata is broken and `erk dash` will not display it correctly. Fix the issue before proceeding.

**CRITICAL: When creating a plan for an objective, ALWAYS use `/erk:objective-plan` to ensure proper metadata linking.** Do not manually reference objectives in plan text without using the structured workflow. The objective-context marker created by this command is required for the plan-save pipeline to link the plan to its parent objective.

### Universal Tripwires

These critical rules apply across all code areas.

@docs/learned/universal-tripwires.md

### Tripwire Routing

Before editing files, load relevant category tripwires.

@docs/learned/tripwires-index.md

**Load these skills FIRST:**

- **Python code** → `dignified-python` skill (LBYL, modern types, ABC interfaces)
- **Test code** → `fake-driven-testing` skill (5-layer architecture, test placement)
- **Dev tools** → Use `devrun` agent (NOT direct Bash for pytest/ty/ruff/prettier/make/gt)

## Core Architecture

**Tech Stack:** Python 3.10+ (uv), Git worktrees, Graphite (gt), GitHub CLI (gh), Claude Code

**Project Structure:**

```
erk/
├── .claude/          # Claude Code commands, skills, hooks
├── .erk/             # Erk configuration, scratch storage
├── docs/learned/     # Agent-generated documentation
├── src/erk/          # Core implementation
└── tests/            # Test suite (5-layer fake-driven architecture)
```

**Design Principles:** Plan-first workflow, worktree isolation, agent-driven development, documentation as code.

## How Agents Work

This file routes to skills and docs; it doesn't contain everything.

**Key Skills** (loaded on-demand):

- `dignified-python`: Python coding standards (LBYL, frozen dataclasses, modern types)
- `fake-driven-testing`: 5-layer test architecture with comprehensive fakes
- `graphite` + `erk-gt`: Graphite stacked PRs (official skill + erk-specific patterns)
- `devrun`: READ-ONLY agent for running pytest/ty/ruff/make

**Documentation Index** (embedded below for ambient awareness):

@docs/learned/index.md

## Planning Workflow

Erk's plan-oriented workflow works across agent backends with different mechanisms.

### Claude Code Users

Key commands:

- `/erk:plan-save` — save plan as draft PR
- `/erk:plan-implement` — implement from a saved plan
- Plan mode and hooks handle lifecycle automatically

### Codex Users

Without built-in plan mode, follow this explicit protocol:

1. **Assess complexity**: For complex tasks (3+ files, unclear scope), create a plan first
2. **Write the plan**: Create a markdown file with implementation steps
3. **Save to GitHub**: Run `erk pr create --file <path-to-plan.md>` to create a tracked draft PR
4. **Implement**: Run `erk implement <plan-number>` to set up a worktree and execute
5. **Submit**: Run `erk pr submit` after implementation to push the code

**Validation coordination**: When implementing, verify that:

- `erk exec impl-init --json` returns `"valid": true` before starting work
- `.erk/impl-context/plan.md` is treated as immutable during implementation
- `erk exec impl-verify` confirms `.erk/impl-context/` is preserved after implementation

### All Backends

- `erk pr list` — view open plans
- `erk implement <plan>` — implement a plan in a worktree

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dagster-io/erk](https://github.com/dagster-io/erk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
