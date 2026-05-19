---
trigger: always_on
description: KTX is a standalone open-source context layer for database agents. These
---

# KTX Development Notes

KTX is a standalone open-source context layer for database agents. These
instructions apply to all agents working in this repository (Codex, Claude,
Gemini, and similar tools). Do not assume an external app server, frontend,
database migrations, ORPC contracts, or `python-service/` layout exist here.

## Critical Rules

### Absolute Requirements

- **MUST**: Use the active agent's task tracker for tasks with 3+ steps or
  complex operations (`TodoWrite` in Claude, `update_plan` in Codex).
- **MUST**: Read files before editing them.
- **MUST**: Complete all tracked tasks before finishing.
- **MUST**: Activate `.venv` before running Python code when a local virtualenv
  exists. If no `.venv` exists, use `uv run ...` from the relevant project root.
- **MUST**: After modifying Python files, run the relevant Python tests and run
  `uv run pre-commit run --files [FILES]` when a pre-commit config exists. If
  pre-commit cannot run because config or tool versions are missing, state that
  explicitly and run the closest available checks.
- **MUST**: Remove dead code; do not leave commented-out code, unused wrappers,
  or empty directories.
- **MUST**: Keep package/public API changes intentional. Do not add compatibility
  wrappers for old KTX names unless the user explicitly asks for a migration
  bridge.
- **MUST**: Treat KTX as having no public users unless the user says otherwise.
  Legacy support is not necessary by default; prefer clean breaking changes over
  compatibility shims, migration bridges, or preserved stale behavior.

### Absolute Prohibitions

- **MUST NOT**: Use raw `pip`; use `uv`.
- **MUST NOT**: Use `npm` or `bun`; use `pnpm`.
- **MUST NOT**: Run destructive git cleanup commands (`git clean`,
  `git reset --hard`, `git checkout .`) unless the user explicitly requested
  that exact operation.
- **MUST NOT**: Run `git stash`, `git stash pop`, `git stash apply`, or
  `git stash drop` without explicit user instruction. Prefer a branch plus
  commit when the user asks to save work in progress.
- **MUST NOT**: Reintroduce external app conventions such as ORPC contracts,
  NestJS controllers, frontend routes, `routeTree.gen.ts`, or app database
  migration commands unless those systems are intentionally added to KTX later.

### Language Convention

- **MUST**: Absolute requirement, never deviate.
- **MUST NOT**: Absolute prohibition.
- **SHOULD**: Strong recommendation, deviate only with good reason.
- **MAY**: Optional, at agent's discretion.

## Priority Hierarchy

When rules conflict, follow this order:

1. Safety and user intent
2. Correctness: code works and verification passes
3. Single source of truth and DRY design
4. Code quality: types, readable boundaries, focused modules
5. Performance where it matters

## Repository Shape

KTX is a pnpm + uv workspace.

- TypeScript packages: `packages/*`
- CLI package: `packages/cli`
- Core context package: `packages/context`
- LLM package: `packages/llm`
- Database connectors: `packages/connector-*`
- Python semantic layer: `python/ktx-sl`
- Python daemon: `python/ktx-daemon`
- Examples and fixtures: `examples/`
- Workspace scripts: `scripts/`
- Local agent skills and internal planning docs are private overlays. Do not
  commit `.agents/`, `.claude/`, or `docs/superpowers/` to this public
  repository.

Some package names still contain `ktx` during the split. Do not mass-rename
symbols, package names, paths, or docs to `ktx` unless the task asks for that
rename.

## Quick Commands

### TypeScript Workspace

```bash
pnpm install
pnpm run build
pnpm run type-check
pnpm run test
pnpm run check
pnpm run dead-code
pnpm --filter @ktx/cli run smoke
pnpm --filter './packages/*' run build
pnpm --filter './packages/*' run test
pnpm --filter './packages/*' run type-check
```

### Python Workspace

```bash
uv sync --all-groups
uv run pytest -q
uv run pytest python/ktx-sl/tests -q
uv run pytest python/ktx-daemon/tests -q
uv run pre-commit run --files [FILES]
```

If `pyproject.toml` pins a newer `uv` than the local binary, do not edit the
pin just to make checks pass. Report the version mismatch and run checks that
do not require changing project configuration.

### CLI and Release Checks

```bash
pnpm run setup:dev
pnpm run link:dev
pnpm run artifacts:verify
pnpm run release:readiness
pnpm run release:published-smoke
```

## Verification After Changes

Choose the smallest checks that cover the changed surface, then broaden when
shared contracts or package exports are affected.

- TypeScript package code: `pnpm --filter <package> run type-check` and
  `pnpm --filter <package> run test`
- Cross-package TypeScript changes: `pnpm run type-check` and `pnpm run test`
- Build/export changes: `pnpm run build`
- Workspace scripts: `node --test scripts/*.test.mjs` or the specific script
  test file
- TypeScript dead-code tooling/config changes: `pnpm run dead-code`
- Python semantic layer: `uv run pytest python/ktx-sl/tests -q`
- Python daemon: `uv run pytest python/ktx-daemon/tests -q`
- Python files: also run `uv run pre-commit run --files [FILES]` when
  pre-commit is configured

For test suites that take a while, capture full output once and inspect that
file instead of rerunning to apply different filters:

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kaelio/ktx](https://github.com/Kaelio/ktx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
