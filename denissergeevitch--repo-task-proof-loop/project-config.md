---
trigger: always_on
description: Repo-local workflow skill for large coding tasks. Initializes .agent/tasks/TASK_ID artifacts, installs project-scoped Codex and Claude subagents, updates AGENTS.md plus the repo's Claude guide file with the workflow, and runs a spec-freeze → build → evidence → verify → fix loop with fresh-session verification.
---


# Repo Task Proof Loop

Use this skill when the user wants a repeatable, auditable implementation workflow for a non-trivial coding task, especially a feature, refactor, migration, or bug fix that should leave repo-local proof in `.agent/tasks/<TASK_ID>/`.

All task artifacts created by this workflow must stay inside the repository.

When the examples below mention `scripts/task_loop.py`, that path is relative to this skill root. Run it while your shell working directory is inside the target repository.

## What this skill does

1. Initializes a strict repo-local task folder under `.agent/tasks/<TASK_ID>/`
2. Seeds or updates the required artifact files
3. Installs project-scoped Codex and Claude subagent templates into `.codex/agents/` and `.claude/agents/`
4. Updates the repo-root `AGENTS.md` Codex baseline plus the repo's Claude guide file (`CLAUDE.md` or `.claude/CLAUDE.md`) with a managed block that explains the workflow
5. Guides the agent through a strict loop:
   - spec freeze
   - builder implementation
   - evidence packing
   - fresh verification
   - minimal fix
   - fresh verification again until `PASS`

See:
- `references/REFERENCE.md`
- `references/COMMANDS.md`
- `references/SUBAGENTS.md`
- `references/SCHEMAS.md`

## Commands this skill supports

Treat the following words as commands when the user invokes this skill:

- `init <TASK_ID>`: create `.agent/tasks/<TASK_ID>/`, install or refresh subagent templates, and update `AGENTS.md` plus the repo's Claude guide file
- `freeze <TASK_ID>`: create or refine `spec.md` from the user task, task file, and repo guidance
- `build <TASK_ID>`: implement the task against the frozen spec
- `evidence <TASK_ID>`: create or refresh `evidence.md`, `evidence.json`, and raw artifacts without changing production code
- `verify <TASK_ID>`: run a fresh verifier pass and write `verdict.json`, plus `problems.md` when needed
- `fix <TASK_ID>`: apply the smallest safe fix set from `problems.md`, then refresh the evidence bundle
- `run <TASK_ID>`: execute the full loop from spec freeze through verification
- `status <TASK_ID>`: summarize current artifact status

If the user does not supply a command, infer the next step from repo state:
- If the task folder does not exist, run `init` first. If the user clearly wants initialization only, stop there. Otherwise, after `init` succeeds and `.agent/tasks/<TASK_ID>/spec.md` exists, continue by re-evaluating repo state in the same turn. Do not overlap `init` with `freeze`, `build`, `evidence`, `verify`, `fix`, `validate`, `status`, or subagent work.
- If `spec.md` is missing or placeholder-only, do `freeze`
- If implementation is not yet complete, do `build`
- If evidence is stale or missing, do `evidence`
- If no fresh verdict exists, do `verify`
- If verdict is not `PASS`, do `fix`

## Initialization step

Run the bundled initializer from the repository root or current working directory inside the repo:

```bash
scripts/task_loop.py init --task-id <TASK_ID>
```

Optional task seeding:

```bash
scripts/task_loop.py init --task-id <TASK_ID> --task-file path/to/task.md
scripts/task_loop.py init --task-id <TASK_ID> --task-text "User task text"
```

The initializer will:

- resolve the repo root
- create `.agent/tasks/<TASK_ID>/`
- create all required artifacts, including placeholders under `raw/`
- install project-scoped subagent files
- insert or refresh managed workflow blocks in `AGENTS.md` and the repo's Claude guide file

For Codex, the initializer keeps its managed workflow block in the repo-root `AGENTS.md`. Codex also supports `AGENTS.override.md` and configured fallback guide filenames; nested files closer to the code still take precedence, and this skill intentionally does not overwrite them.
If `init` creates or rewrites `AGENTS.md` during a running Codex session, start a new Codex session before relying on the updated instructions. Codex snapshots project-doc guidance at session start.

For Claude Code, the initializer keeps its managed workflow block in the repo-root `CLAUDE.md`. Claude Code also supports `.claude/CLAUDE.md`, `.claude/rules/*.md`, and `CLAUDE.local.md`, but this skill treats root `CLAUDE.md` as the primary project guide because Claude surfaces it directly.

In Claude Code, if `init` just wrote or refreshed `.claude/agents/*` during the current session, do not assume those updated agents are already available mid-session.

Treat `init` as a serial prerequisite. Never overlap it with `freeze`, `build`, `evidence`, `verify`, `fix`, `validate`, `status`, or child-agent spawning.

## Heavy-task default workflow

For large tasks, keep the user-facing request simple. In Codex, continue serially unless the user explicitly asks for delegation or parallel agent work; after that authorization, the skill can choose the internal child setup automatically when the current product surface supports delegation and the task shape warrants it.

### Preferred delegated sequence


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DenisSergeevitch/repo-task-proof-loop](https://github.com/DenisSergeevitch/repo-task-proof-loop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
