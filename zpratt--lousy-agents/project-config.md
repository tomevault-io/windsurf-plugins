---
trigger: always_on
description: **Mandatory**: Always read [](./context/project.context.md) for full project context before contributing.
---


# Project Overview

**Mandatory**: Always read [](./context/project.context.md) for full project context before contributing.

## Shared Instruction Architecture

This repo serves instructions to both GitHub Copilot and Claude Code from a single set of canonical files, organized so each topic lives in exactly one place:

- **This file (`.github/copilot-instructions.md`)** is the canonical home for **repo-wide general guidance** (commands, TDD workflow, tech stack, project structure, code style, dependencies, boundaries, task tracking). Copilot code review always loads it; Claude Code imports it from the root `CLAUDE.md`.
- **`.github/instructions/*.instructions.md`** are the canonical home for **scoped domain rules** (testing, software architecture, specs, pipelines). Each has an `applyTo` glob. Copilot code review auto-applies them to matching changed files; Claude Code imports them from nested `CLAUDE.md` files placed in the matching directories. The general sections in this file deliberately summarize (not duplicate) those rules and link to them.

`CLAUDE.md` files use `@path/to/file` syntax (e.g., `@.github/copilot-instructions.md`) to import these shared files. This is Claude Code's native file reference mechanism — **not** a broken markdown link. Do not suggest converting `@path/to/file` references in any `CLAUDE.md` to markdown links. Copilot does **not** follow `@import` or markdown links, so anything Copilot review must see is kept physically in this file or in `.github/instructions/*`.

## Commands

Mise manages all tools and Node versions. If you haven't activated mise in your shell, run `mise activate` once or prefix commands with `mise exec --`. During development, use file-scoped commands for faster feedback, and run the full validation suite (`mise run ci`) before commits.

```bash
# One-time shell setup (or add to ~/.zshrc)
eval "$(mise activate zsh)"

# Core commands
mise run test            # Run tests (vitest)
mise run lint            # Run ALL linting tools in parallel (Biome, actionlint, yamllint, markdownlint, shellcheck, semgrep, dependency-cruiser, issue-form schemas)
mise run format-check    # Biome only — code formatting + static analysis
mise run format-fix      # Auto-fix Biome lint/format issues
npm run build            # Production build

# Workspace-scoped commands
npm run build --workspace=packages/cli     # Build CLI only
npm test -- packages/cli/src               # Test files in a specific package path

# File-scoped (faster feedback)
npx biome check path/to/file.ts
npm test path/to/file.test.ts

# Validation suite (run before commits)
mise run ci              # Runs: lint -> test -> test-integration -> smoke-test (test-integration and smoke-test both depend on build)

# Other
npm audit                # Security check
npm install              # Install deps (updates package-lock.json)
```

**Note**: In GitHub Actions, `jdx/mise-action` automatically activates mise and makes all tools available in PATH. No additional setup needed in CI.

This repository is an npm workspace monorepo. Run `npm install` once at the root to install all workspace dependencies. The root `npm run build` command builds the publishable packages: `packages/cli`, `packages/mcp`, and `packages/agent-shell`.

## Workflow: TDD Required

Follow this exact sequence for ALL code changes. Work in small increments — make one change at a time and validate before proceeding.

1. **Research**: Search codebase for existing patterns, components, utilities. Use Context7 MCP tools for library/API documentation.
2. **Write failing test**: Create test describing desired behavior
3. **Verify failure**: Run `mise run test` — confirm clear failure message
4. **Implement minimal code**: Write just enough to pass
5. **Verify pass**: Run `mise run test` — confirm pass
6. **Refactor**: Clean up, remove duplication, keep tests green
7. **Verify refactor**: Run `mise run test && mise run lint` — confirm tests still green and all linting passes
8. **Validate**: `mise run ci` — runs `lint -> test -> test-integration -> smoke-test`

Task is NOT complete until `mise run ci` exits 0.

## Tech Stack

- **Framework**: CLI using c12 for configuration management and citty for terminal interactions
  - When choosing additional libraries, prefer ones that integrate well with c12/citty (from [UnJS ecosystem](https://unjs.io/))
- **Language**: TypeScript (strict mode)
- **Validation**: Zod for runtime validation of external data
- **Testing**: Vitest (never Jest), MSW for HTTP mocking, Chance.js for test fixtures
- **Linting**: Biome (never ESLint/Prettier separately)
- **Logging**: Consola with JSON format and child loggers
- **HTTP**: fetch API only
- **Architecture**: Clean Architecture principles

## Project Structure

```
.github/           GitHub Actions workflows
packages/
  core/            Shared domain entities, use cases, gateways, and formatters
  cli/             Published CLI package and reference scaffold templates
  mcp/             Published MCP server package
  action/          Private GitHub Action package
  agent-shell/     Published npm script-shell telemetry package
scripts/           Build, deploy, and test scripts
.nvmrc             Node.js version (latest LTS)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zpratt/lousy-agents](https://github.com/zpratt/lousy-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
