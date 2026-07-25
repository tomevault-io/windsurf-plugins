---
trigger: always_on
description: DevBuddy (`bud`) is a development environment manager. It reads a `dev.yml` file in a project root and automates environment setup (language versions, virtualenvs, system packages) and provides custom project commands. It integrates deeply with the user's shell (bash/zsh) via a shell hook that automatically activates/deactivates environments as you `cd` between projects.
---

# DevBuddy - Claude Instructions

## Overview

DevBuddy (`bud`) is a development environment manager. It reads a `dev.yml` file in a project root and automates environment setup (language versions, virtualenvs, system packages) and provides custom project commands. It integrates deeply with the user's shell (bash/zsh) via a shell hook that automatically activates/deactivates environments as you `cd` between projects.

### Purpose & scope

DevBuddy is an open-source equivalent of Shopify's internal `dev` tool. The goal is to keep a comparable scope to `dev`: manage the **whole developer environment**, not just language versions. That includes:

- Language toolchains and project virtualenvs (Go, Python, Node, Ruby, ...)
- System packages (Homebrew, apt) and platform-level prerequisites (e.g. accepting the Xcode license after an OS upgrade)
- Project ergonomics: cloning, `cd`ing between repos, opening project URLs, project-local commands
- Future: worktree management and other repo-level workflows

This broader scope is intentional and is what differentiates DevBuddy from pure version managers.

### Related projects (reference material)

Two well-maintained projects overlap with DevBuddy on the version-manager axis and are good references when designing language tasks:

- **asdf** — https://github.com/asdf-vm/asdf — pluggable, single-tool version manager; `.tool-versions` file; rich plugin ecosystem.
- **mise** — https://github.com/jdx/mise — Rust-based asdf-compatible version manager; also handles env vars and project tasks; very active.

When adding or revising language support, check how asdf/mise handle the same case (version-file conventions, plugin/install flow, shimming vs PATH activation, lockfile watching). DevBuddy's *scope* is wider than theirs, but its *version-manager surface* should not be worse than theirs for the languages it supports.

## Roadmap

See `ROADMAP.md` for the product and technical direction. Consult it before making architectural changes to ensure they align with the desired direction. Key points:
- Moving toward a **runtime environment abstraction** (command execution, filesystem, terminal I/O, env vars, state store) with real and testing implementations
- Near-term: refactor `pkg/executor/` from interface+builder to a plain struct representing a command request
- Changes should move toward testability without Docker/PTY and away from direct OS calls in business logic

## Planning

Write implementation plans in `docs/plans/`. Keep plans concrete enough to execute incrementally, with file paths, validation commands, and PR-sized steps.

## Product Design Notes

When the user gives product design directives or user-facing behavior guidelines, record them in `docs/design.md` as part of the development process. Keep the document concise and occasionally consolidate it by reviewing existing notes, removing duplication, and resolving ambiguity.

## Project Structure

```
cmd/bud/main.go          # Entry point. Version set via build flags (-ldflags)
pkg/
  cmd/                    # Cobra CLI commands (root, up, cd, clone, open, create, init, inspect, upgrade, commands)
  config/                 # Global config (debug mode via BUD_DEBUG env var)
  context/                # Runtime context: loads project, env, UI
  env/                    # Environment variable mutation tracking (set, prepend, compare)
  executor/               # Command execution helpers
  autoenv/                # Feature activation/deactivation state machine
    feature_info.go       # FeatureInfo (name+param), FeatureSet
    state.go              # Persisted state of active features (JSON in BUD_AUTO_ENV_FEATURES)
    runner.go             # Sync() - activates/deactivates features by diffing desired vs current
  hook/                   # Shell hook logic (called on every prompt via `bud --shell-hook`)
  integration/            # Shell integration scripts (embedded via //go:embed)
    common.sh             # bud() wrapper, __bud_prompt_command, finalizers
    bash.sh               # PROMPT_COMMAND hook
    zsh.sh                # precmd_functions hook
    integration.go        # Print(), CompletionScriptProvider, AddFinalizerCd
  manifest/               # dev.yml parsing
  project/                # Project discovery (walks up to find dev.yml)
  tasks/                  # Task implementations
    api/                  # TaskAction interface, Condition interface (file checksum tracking)
    taskengine/           # Task execution engine
    golang.go             # Go version management (installs Go, sets GOROOT/GOPATH/GO111MODULE)
    python.go             # Python via pyenv
    python_develop.go     # pip install -e . (tracks setup.py/pyproject.toml changes)
    pip.go                # pip install -r requirements.txt
    pipfile.go            # Pipfile support
    node.go               # Node.js version management
    homebrew.go           # Homebrew package installation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devbuddy/devbuddy](https://github.com/devbuddy/devbuddy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
