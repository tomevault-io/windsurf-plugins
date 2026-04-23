---
trigger: always_on
description: This is the **north star** for AI coding agents working on Taskplane.
---

# AGENTS.md

## Purpose

This is the **north star** for AI coding agents working on Taskplane.

Taskplane is an experimental but production-minded pi package for:
- single-task autonomous execution (`/task`)
- dependency-aware parallel orchestration (`/orch*`)
- file-backed state, resumability, and observability

When in doubt, optimize for: **determinism, recoverability, and clear operator visibility**.

---

## Project map (what to read first)

### 1) Global orientation
1. `README.md` (user-facing behavior)
2. `docs/README.md` (full docs map)
3. `docs/explanation/architecture.md`

### 2) If your change is about `/orch*`
- `extensions/taskplane/extension.ts` (command surface)
- `extensions/taskplane/discovery.ts` (task discovery + deps)
- `extensions/taskplane/waves.ts` (DAG/waves/assignment)
- `extensions/taskplane/execution.ts` (lane execution)
- `extensions/taskplane/merge.ts` (merge flow)
- `extensions/taskplane/persistence.ts` + `resume.ts` (resume/state)
- `extensions/taskplane/types.ts` (defaults + contracts)
- `docs/reference/commands.md`
- `docs/reference/configuration/task-orchestrator.yaml.md`

### 3) If your change is about CLI/dashboard/scaffolding
- CLI: `bin/taskplane.mjs`
- Dashboard: `dashboard/server.cjs`, `dashboard/public/*`
- Templates: `templates/**`
- Packaging: `package.json`, `docs/maintainers/package-layout.md`

### 4) Tests
- `extensions/tests/*`
- `docs/maintainers/testing.md`

---

## Core architecture invariants (do not break casually)

1. **File-backed execution memory is fundamental**
   - `STATUS.md` is persistent task memory.
   - `.DONE` is authoritative completion marker.

2. **Orchestrator state must be resumable**
   - Persisted state in `.pi/batch-state.json` is part of runtime contract.
   - Resume/abort flows depend on consistent state semantics.

3. **Task execution and orchestration are separate concerns**
   - `/orch*` behavior coordinates discovery/waves/lanes/worktrees/merge.

4. **Templates are public scaffolding, not project-specific policy**
   - Keep template examples generic and safe for open-source distribution.

5. **Published package boundaries matter**
   - Only files in `package.json#files` ship.
   - Changes to package layout or manifest impact install/runtime behavior.

6. **Configuration lives in `taskplane-config.json`, not YAML**
   - The canonical project config file is `.pi/taskplane-config.json` (JSON, camelCase keys).
   - Legacy `.pi/task-runner.yaml` and `.pi/task-orchestrator.yaml` files may still exist but are **fallback only** — the JSON config takes precedence when present.
   - When reading or modifying configuration, always check for `taskplane-config.json` first.
   - When documenting config changes, reference the JSON format and keys (e.g., `taskRunner.reviewer.thinking`, not `reviewer:\n  thinking:`).
   - User preferences live in `~/.pi/agent/taskplane/preferences.json` and override project config.
   - See `extensions/taskplane/config-loader.ts` and `extensions/taskplane/config-schema.ts` for the loading chain and defaults.

---

## Always do

1. **Read before editing**
   - Inspect relevant code paths + reference docs before making changes.

2. **Keep behavior and docs aligned**
   - If command/config/format behavior changes, update docs in the same change.

3. **Add or update tests for behavior changes**
   - Especially for discovery, waves, persistence/resume, and command parsing.

4. **Run validations locally (minimum)**
   - `cd extensions && node --experimental-strip-types --experimental-test-module-mocks --no-warnings --import ./tests/loader.mjs --test tests/*.test.ts`
   - If CLI changed: `node bin/taskplane.mjs help` and `node bin/taskplane.mjs doctor`

5. **Preserve compatibility intentionally**
   - If changing external contracts (commands, config keys, state schema), do it explicitly and document it.

6. **Keep commits scoped and reviewable**
   - Separate docs, templates, and runtime logic where possible.

7. **Prefer small, deterministic changes**
   - Avoid broad refactors unless required by the task.

---

## Never do

1. **Never `git reset --hard` when you have uncommitted or staged changes.** Use `git stash` first, or commit to a branch. Hard reset silently destroys work that must then be re-applied from scratch.
2. **Never hardcode machine/user-specific paths or private environment assumptions.**
2. **Never leak internal/planning artifacts into public docs/templates.**
3. **Never make template content project- or language-specific.**
4. **Never silently change command names/flags or config schema fields.**
5. **Never break persistence/resume semantics without schema + docs + tests updates.**
6. **Never bypass `.DONE` / `STATUS.md` conventions in task execution flow.**
7. **Never introduce unnecessary build/runtime complexity for dashboard or extensions.**
8. **Never publish/release as part of routine code edits unless explicitly requested.**

---

## Change checklists by area

### Command behavior changes (`/task`, `/orch*`, CLI)
- Update implementation
- Update `docs/reference/commands.md`
- Update README command tables if needed
- Add/adjust tests

### Config changes (`task-runner.yaml` / `task-orchestrator.yaml`)
- Update defaults/types/loaders in code

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HenryLach/taskplane](https://github.com/HenryLach/taskplane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
