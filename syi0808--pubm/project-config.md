---
trigger: always_on
description: This file guides coding agents when working in this repository.
---

# AGENTS.md

This file guides coding agents when working in this repository.

## Project Overview

pubm is a CLI tool for publishing packages to multiple registries (npm, jsr, and private registries) simultaneously. It supports interactive prompts in TTY mode and CI environments, with automatic rollback on failure.

The repository uses Turborepo and Bun workspaces.

## Repository Layout

```
packages/
  core/                            — @pubm/core: Core SDK (ecosystem, registry, changeset, monorepo, plugin, config, tasks, validate, prerelease, git, utils)
  pubm/                            — pubm: CLI using Commander, depends on @pubm/core
    platforms/                     — Cross-platform binaries
  plugins/
    plugin-brew/                   — @pubm/plugin-brew: Homebrew formula publishing
    plugin-external-version-sync/  — @pubm/plugin-external-version-sync: Syncs version to external files
plugins/
  pubm-plugin/                     — Claude Code plugin (skills for publish, setup, version-sync, etc.)
website/                           — Astro-based documentation site
docs/                              — Planning documents and strategy docs
Formula/                           — Homebrew formula
patches/                           — Dependency patches (listr2)
```

## Commands

Run from the repo root. Turborepo fans out to all packages:

```bash
bun run build          # Build all packages (via turbo)
bun run check          # Lint and format check (via turbo)
bun run format         # Auto-fix lint and formatting issues (biome check --write)
bun run typecheck      # TypeScript type checking (via turbo)
bun run test           # Run all tests (via turbo)
bun run coverage       # Run tests with coverage (via turbo)
bun run dev:site       # Start Astro documentation dev server
bun run build:site     # Build static documentation site
bun run release        # Release with preflight checks
bun run release:ci     # Release in CI environment
bun run changesets:add # Add changesets (via pubm)
```

Run a single test file (within a package):
```bash
cd packages/core && bun vitest --run tests/unit/utils/rollback.test.ts
```

Tests live in `tests/unit/` and `tests/e2e/` within each package. Coverage thresholds are 95% lines/functions/statements and 90% branches. Tests run in the `forks` pool with a 30s timeout.

## Architecture

> See [ARCHITECTURE.md](ARCHITECTURE.md) for architecture diagrams, design patterns, and module organization.
>
> ARCHITECTURE.md is large. Do NOT read the entire file. Instead, use a **subagent(haiku)** to read and summarize only the relevant section. Example:
> ```
> Agent(model: "haiku", prompt: "Read ARCHITECTURE.md and summarize the Registry Abstraction section. Focus on ...")
> ```

## Pre-commit Checklist

Before committing, run these checks in order and fix any failures:

1. `bun run format`: auto-fix lint and formatting issues
2. `bun run typecheck`: ensure no type errors
3. `bun run test`: ensure all tests pass
4. `bun run coverage`: keep coverage thresholds from dropping

For final, task-complete commits, commit only after all four pass.

## Commit Workflow

- Make WIP commits at useful checkpoints when the work is in an executable state or when progress should be preserved.
- Do not wait until the entire task is complete if there is a coherent, runnable unit worth saving.
- Keep WIP commits narrowly scoped to the current task and use commit messages that make the checkpoint status clear.
- WIP commits do not replace final verification. Before a final handoff or final commit, run the required checks from the pre-commit checklist when feasible.

## Coverage Maintenance

Coverage thresholds are enforced per package in `vitest.config.mts`. **Never lower them.**

- Every new source file must have a matching test file in `tests/unit/`
- New code must not decrease coverage: run `bun run coverage` before committing
- Use `/* istanbul ignore next */` sparingly: only for genuinely untestable code such as compile-time constants or exhaustiveness guards. Always include a reason comment
- When modifying a file below threshold, add tests to bring it up

## Changeset Workflow

After completing a bug fix or feature addition, create a changeset:

```bash
bun run changesets:add --packages <package-path> --bump <patch|minor|major> --message "description of the change"
```

- `patch`: bug fixes, minor corrections
- `minor`: new features, non-breaking additions
- `major`: breaking changes

Changesets are required for any user-facing change. Do not commit without adding a changeset when applicable.

### Changeset rules

- **Identifier**: Use the package's filesystem path (e.g., `packages/core`), not the registry name. Package names are also accepted and auto-resolved to paths.
- **Message**: Write in English, from the user's perspective. Describe what changed, not how it was implemented internally.
- **Scope**: Document user-facing changes only. Internal refactors without behavioral impact do not need a changeset.
- **Change basis**: Write the changeset for the change you made in the current task. Do not summarize every change that happened on the branch.

## Documentation Maintenance


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [syi0808/pubm](https://github.com/syi0808/pubm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
