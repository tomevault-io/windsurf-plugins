---
trigger: always_on
description: Guidance for AI coding agents working with this repository.
---

# AGENTS.md

Guidance for AI coding agents working with this repository.

## Prerequisites

- **Go 1.25+**: Check with `go version`
- **AI coding agent CLI**: Authenticated and configured
- **Make, golangci-lint**: For build/lint (`make lint`)

## Commands

```bash
# Build & Dev
make build          # Build for current platform
make test           # Run all tests (quiet, shows failures only)
make test-v         # Run all tests (verbose, for debugging)
make fmt            # Format Go code (run before committing)
make lint           # Run all linters

# Single test
go test -run TestName ./internal/package/

# CLI usage (run `autospec --help` for full reference)
autospec run -a "feature description"    # All stages: specify → plan → tasks → implement
autospec prep "feature description"      # Planning only: specify → plan → tasks
autospec implement --phases              # Each phase in separate session
autospec implement --tasks               # Each task in separate session
autospec st                              # Show status and task progress
autospec doctor                          # Check dependencies
```

## Core Workflow

### Stage Dependencies (MUST follow this order)

```
constitution → specify → plan → tasks → implement
     ↓            ↓        ↓       ↓
constitution.yaml spec.yaml plan.yaml tasks.yaml
```

| Stage | Requires | Produces |
|-------|----------|----------|
| `constitution` | — | `.autospec/constitution.yaml` |
| `specify` | constitution | `specs/NNN-feature/spec.yaml` |
| `plan` | spec.yaml | `plan.yaml` |
| `tasks` | plan.yaml | `tasks.yaml` |
| `implement` | tasks.yaml | code changes |

**Constitution is REQUIRED before any workflow stage.**

### What `autospec init` Does

1. Creates config (`~/.config/autospec/config.yml` or `.autospec/config.yml`)
2. Installs slash commands to agent's command directory (e.g., `.claude/commands/`)
3. Configures agent permissions and sandbox settings
4. Prompts for constitution creation (one-time per project)

### First-Time Project Setup

```bash
autospec init              # Interactive setup (config + agent + constitution)
autospec doctor            # Verify dependencies
autospec prep "feature"    # specify → plan → tasks
autospec implement         # Execute tasks
```

## Documentation

**Review relevant docs before implementation:**

### Internal (Developer-Focused)

| File | Purpose |
|------|---------|
| `docs/internal/architecture.md` | System design, component diagrams, execution flows |
| `docs/internal/go-best-practices.md` | Go conventions, naming, error handling patterns |
| `docs/internal/internals.md` | Spec detection, validation, retry system, phase context |
| `docs/internal/YAML-STRUCTURED-OUTPUT.md` | YAML artifact schemas and slash commands |
| `docs/internal/testing-mocks.md` | Mock testing infrastructure for workflows without real API calls |
| `docs/internal/events.md` | Event-driven architecture using kelindar/event |
| `docs/internal/risks.md` | Risk documentation in plan.yaml |
| `docs/internal/cclean.md` | claude-clean tool for transforming streaming JSON output |

### Public (User-Focused)

| File | Purpose |
|------|---------|
| `docs/public/reference.md` | Complete CLI command reference with all flags |
| `docs/public/quickstart.md` | Getting started guide for first workflow |
| `docs/public/overview.md` | High-level introduction to autospec |
| `docs/public/TIMEOUT.md` | Timeout configuration and behavior |
| `docs/public/checklists.md` | Checklist generation, validation, and implementation gating |
| `docs/public/SHELL-COMPLETION.md` | Shell completion implementation |
| `docs/public/troubleshooting.md` | Common issues and solutions |
| `docs/public/claude-settings.md` | Claude Code settings and sandboxing configuration |
| `docs/public/opencode-settings.md` | OpenCode configuration, permissions, and command patterns |
| `docs/public/agents.md` | CLI agent configuration (Claude and OpenCode supported) |
| `docs/public/worktree.md` | Git worktree management for parallel agent execution |
| `docs/public/parallel-execution.md` | Sequential and parallel execution with DAG scheduling |
| `docs/public/self-update.md` | Version checking and self-update functionality |
| `docs/public/faq.md` | Frequently asked questions |

### Research

| File | Purpose |
|------|---------|
| `docs/research/claude-opus-4.5-context-performance.md` | Claude Opus 4.5 performance in extended sessions |

## Architecture Overview

autospec is a Go CLI that orchestrates SpecKit workflows. Key distinction:
- **Stage**: High-level workflow step (specify, plan, tasks, implement)
- **Phase**: Task grouping within implementation (Phase 1: Setup, Phase 2: Core, etc.)

### Package Structure

- `cmd/autospec/main.go`: Entry point
- `internal/cli/`: Cobra commands (root + orchestration)
  - `internal/cli/stages/`: Stage commands (specify, plan, tasks, implement)
  - `internal/cli/config/`: Configuration commands (init, config, migrate, doctor)
  - `internal/cli/util/`: Utility commands (status, history, version, clean, view)
  - `internal/cli/admin/`: Admin commands (commands, completion, uninstall)
  - `internal/cli/worktree/`: Worktree management commands (create, list, remove, prune)
  - `internal/cli/shared/`: Shared types and constants

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ariel-frischer/autospec](https://github.com/ariel-frischer/autospec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
