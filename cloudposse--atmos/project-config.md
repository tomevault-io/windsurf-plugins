---
trigger: always_on
description: Guidance for Claude Code when working with this repository.
---

# CLAUDE.md

Guidance for Claude Code when working with this repository.

## Project Overview

Atmos: Go CLI for cloud infrastructure orchestration via Terraform/Helmfile/Packer with stack-based config, templating, policy validation, vendoring, and terminal UI.

## Git Worktrees (MANDATORY)

This repository uses git worktrees for parallel development. When working in a worktree:

- **ALWAYS stay within the current working directory** - Never escape to parent directories
- **Use relative paths or paths under the current working directory** - Do not hardcode `/Users/*/atmos/` paths
- **The worktree IS the repository** - All files, including `pkg/`, `cmd/`, `internal/`, exist within the worktree
- **Never assume the parent directory is the repo** - Worktrees like `.conductor/branch-name/` are complete, independent working copies

**Why this matters:** Searching outside the worktree will find stale code from the main branch instead of the current branch's code. This leads to incorrect analysis and recommendations.

**For Task agents:** When searching for files, always use the current working directory (`.`) or relative paths. Never construct absolute paths that might escape the worktree.

## Concurrent Sessions (MANDATORY)

Multiple Claude sessions may be working on the same branch or worktree simultaneously. To avoid destroying other sessions' work:

- **NEVER delete, reset, or discard files you didn't create** - Other sessions may have created them
- **NEVER run `git reset`, `git checkout --`, or `git clean`** without explicit user approval
- **NEVER run `go clean`**. In particular, `go clean -cache` deletes the shared Go build cache and breaks concurrent builds; it is not an approved troubleshooting step.
- **ALWAYS ask the user before removing untracked files** - They may be work-in-progress from another session
- **When you see unfamiliar files**, assume another session created them - ask the user what to do
- **If pre-commit hooks fail due to files you didn't touch**, ask the user how to proceed rather than trying to fix or remove them

**Why this matters:** The user may have multiple Claude sessions working in parallel on different aspects of a feature. Deleting “unknown” files destroys that work.

## Hourly PR Maintenance Loop (RECOMMENDED)

On a branch with an open PR, use the **`pr-maintenance-loop`** skill (`.claude/skills/pr-maintenance-loop/SKILL.md`) to start an hourly `/loop` that works the PR toward merge-ready: rebases against `main` when behind, checks CI, addresses and resolves unresolved CodeRabbit threads, and runs the patch-scoped `lint` and `test-coverage` skills. This loop is session-only — it dies with the process and expires after 7 days — so re-invoke the skill each new session; it is not a one-time setup. For a single on-demand pass without starting a recurring loop, invoke the **`fix-all`** skill directly (also mirrored at the CLI as `atmos fix --all`).

## Essential Commands

```bash
# Build & Test
atmos build                  # Build to ./build/atmos
atmos test                   # Run short tests
atmos test --full            # Run full acceptance tests
atmos test --coverage        # Tests with coverage
atmos lint --changed         # golangci-lint on changed files
```

## Architecture

- **`cmd/`** - CLI commands (one per file)
- **`internal/exec/`** - Business logic
- **`pkg/`** - config, stack, component, utils, validate, workflow, hooks, telemetry

**Stack Pipeline**: Load atmos.yaml → process imports/inheritance → apply overrides → render templates → generate config.

**Templates and YAML functions**: Go templates + Gomplate with `atmos.Component()`, `!terraform.state`, `!terraform.output`, store integration.

## Working with Atmos Agents (RECOMMENDED)

Atmos has **specialized domain experts** in `.claude/agents/` for focused subsystems. **Use agents instead of inline work** for their areas of expertise.

**Available Agents:**
- **`@agent-developer`** - Creating/maintaining agents, agent architecture
- **`@tui-expert`** - Terminal UI, theme system, output formatting
- **`@atmos-errors`** - Error handling patterns, error builder usage
- **`@flag-handler`** - CLI commands, flag parsing, CommandProvider pattern
- **`@example-creator`** - Creating examples, mock components, test cases, EmbedFile docs

**When to delegate:**
- TUI/theme changes → `@tui-expert`
- New CLI commands → `@flag-handler`
- Error handling refactoring → `@atmos-errors`
- Creating new agents → `@agent-developer`
- Creating examples/demos → `@example-creator`

**Benefits:** Agents are domain experts with deep knowledge of patterns, PRDs, and subsystem architecture. They ensure consistency and best practices.

See `.claude/agents/README.md` for full list and `docs/prd/claude-agent-architecture.md` for architecture.

## Architectural Patterns (MANDATORY)

### Registry Pattern (MANDATORY)
Use registry pattern for extensibility. Existing implementations:
- **Command Registry**: `cmd/internal/registry.go` - All commands register via `CommandProvider` interface
- **Store Registry**: `pkg/store/registry.go` - Multi-provider store implementations

**New commands MUST use command registry pattern.** See `docs/prd/command-registry-pattern.md`

### Interface-Driven Design (MANDATORY)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudposse/atmos](https://github.com/cloudposse/atmos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
