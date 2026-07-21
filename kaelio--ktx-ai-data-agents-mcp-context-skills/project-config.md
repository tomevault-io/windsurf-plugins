---
trigger: always_on
description: **ktx** is a standalone open-source context layer for data agents. These
---

# ktx Development Notes

**ktx** is a standalone open-source context layer for data agents. These
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
  wrappers for old **ktx** names unless the user explicitly asks for a migration
  bridge.
- **MUST**: Avoid compatibility shims for old **ktx** features, command shapes,
  configuration formats, or internal APIs. This rule does not prohibit
  compatibility support for third-party systems and libraries, such as
  Metabase version differences. Keep the **ktx** codebase clean instead of
  preserving stale **ktx** behavior.
- **MUST**: Treat **ktx** as having no public users unless the user says otherwise.
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
  migration commands unless those systems are intentionally added to **ktx** later.

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

## Opinionated Product Defaults

- **MUST**: Prefer one canonical behavior over configurable alternatives. A new
  flag, config field, environment variable, mode, strategy option, adapter hook,
  or fallback path is a product feature and must be justified by an explicit
  user request or a real correctness requirement.
- **MUST NOT**: Add speculative flexibility for imagined users, migrations,
  review preferences, local workflows, or "just in case" scenarios. If the
  requested behavior can work with one solid default, implement that default.
- **MUST NOT**: Add boolean switches that create two runtime paths unless both
  paths are essential and the user explicitly asked for the choice. Boolean
  policy knobs are especially suspect because they double the state space and
  test surface.
- **MUST**: When a design seems to need a new option, first try to remove the
  need by choosing the stronger default, tightening the invariant, or failing
  clearly. Ask the user before adding the option if it still seems necessary.
- **MUST**: Delete obsolete branches, tests, docs, and config when removing a
  behavior. Do not preserve dormant compatibility paths.

## Repository Shape

**ktx** is a pnpm + uv workspace.

- TypeScript package: `packages/cli` (the sole npm-published package source)
- Core context modules: `packages/cli/src/context/`
- LLM provider modules: `packages/cli/src/llm/`
- Database connector modules: `packages/cli/src/connectors/<driver>/`
- Python semantic layer: `python/ktx-sl`
- **ktx** daemon: `python/ktx-daemon`
- Examples and fixtures: `examples/`
- Workspace scripts: `scripts/`
- Local agent skills and internal planning docs are private overlays. Do not
  commit `.agents/`, `.claude/`, or `docs/superpowers/` to this public
  repository.

Some source identifiers still contain historical package-oriented names. Do not
mass-rename symbols, paths, or docs unless the task asks for that rename.

## Quick Commands

### TypeScript Workspace

```bash
pnpm install
pnpm run build
pnpm run type-check
pnpm run test
pnpm run check
pnpm run dead-code
pnpm --filter @kaelio/ktx run smoke

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kaelio/ktx-ai-data-agents-mcp-context-skills](https://github.com/Kaelio/ktx-ai-data-agents-mcp-context-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
