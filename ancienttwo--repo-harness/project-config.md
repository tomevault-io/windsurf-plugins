---
trigger: always_on
description: This repository self-hosts the `repo-harness` contract; the former `repo-harness-skill` and `project-initializer` names are retired and removed by installed-copy sync. Claude and Codex should follow the same repo-local workflow surface.
---

# repo-harness AGENTS.md

This repository self-hosts the `repo-harness` contract; the former `repo-harness-skill` and `project-initializer` names are retired and removed by installed-copy sync. Claude and Codex should follow the same repo-local workflow surface.

## Canonical Workflow Files

- `tasks/current.md` for the tracked current-status snapshot derived from workflow artifacts
- `tasks/todos.md` for deferred medium/long-term goals, not active execution checklists
- `plans/prds/` for upper-layer PRDs; `plans/sprints/` for ordered sprint backlogs operated through `scripts/sprint-backlog.sh` (installed implementations live under `.ai/harness/scripts/`; task contracts stay the execution slices)
- `.ai/context/capabilities.json` for the capability registry and longest-prefix context boundaries
- `tasks/workstreams/` for capability long-running workstreams that project durable progress into local contracts
- `tasks/lessons.md` for correction-derived rules
- `docs/researches/` for deep repo knowledge
- `tasks/notes/` for task-local implementation decisions, deviations, tradeoffs, and open questions
- `plans/` for timestamped plans, with `plans/archive/` for history
- `.ai/harness/workflow-contract.json` for the installed workflow contract manifest
- `.ai/harness/policy.json` for the machine-readable workflow contract
- `.ai/context/context-map.json` for progressive context loading
- `docs/architecture/index.md` for umbrella architecture status, drift requests, snapshots, and diagram links
- `docs/reference-configs/agentic-development-flow.md` for gstack/Waza routing rules

## Operating Rules

- Sync `tasks/` whenever substantive repo changes are made.
- Use `tasks/notes/<plan-stem>.notes.md` only for non-obvious slice decisions, deviations, tradeoffs, and open questions; `<plan-stem>` is the active plan filename without `plan-` and `.md` (for example `20260531-0045-governance-workflow`). Do not use notes as durable memory or a task log, and archive/promote them deliberately when the slice closes.
- Treat hook execution as central-first: trusted repos run `~/.repo-harness/hooks/` (bash shim) or the packaged CLI copy; this self-host repo pins `"hook_source": "repo"` in `.ai/harness/policy.json` so `.ai/hooks/` stays the live development runtime, with `assets/hooks/` as the product source mirrored on install. User-level `~/.claude/settings.json` and `~/.codex/hooks.json` are the host adapters.
- Keep the umbrella hierarchy explicit: architecture owns stable truth, capability contracts own local agent context, `tasks/workstreams/<domain>/<capability>/` owns durable progress, and `tasks/todos.md` owns only deferred medium/long-term goals with tradeoff and revisit trigger.
- Treat `.ai/context/capabilities.json` as the source of truth for capability prefixes; `agent-context-blocks.txt` and nested agent files are compatibility inputs only.
- Keep architecture drift handling split: `architecture-queue.sh` writes architecture requests/events, `workstream-sync.sh` maintains durable capability workstreams, and `context-contract-sync.sh` only updates controlled local `CLAUDE.md`/`AGENTS.md` architecture blocks.
- Keep `assets/workflow-contract.v1.json` and `.ai/harness/workflow-contract.json` in sync.
- Keep `CLAUDE.md` and `AGENTS.md` short; put detailed guidance in `docs/reference-configs/`.
- Treat Codex auto-compact as a fallback only; use `.ai/harness/handoff/current.md` and `.ai/harness/handoff/resume.md` for long-task rollover.
- Treat `_ref/` as an occasional ignored external reference checkout cache, not a commit surface or daily workflow. Agents may read or refresh it for comparison; when it influences a decision, cite the source repo plus commit/tag and path in `tasks/notes/` or `docs/researches/`.
- Treat `deploy/` as the trackable deployment and operations surface for runbooks, submission materials, release checklists, helper scripts, ordered SQL files under `deploy/sql/`, and env examples.
- Treat `_ops/` as ignored local operations state for secrets, real env files, provider state, artifacts, logs, and scratch files; do not commit or agent-edit `_ops/*`.
- Treat contract-level task execution as worktree-first: `scripts/plan-to-todo.sh --plan <approved-plan>` starts `scripts/contract-worktree.sh start --plan <approved-plan>` when policy enables it, and completed blocks finish through Waza `/check` plus `scripts/contract-worktree.sh finish`.
- After Codex Plan mode, Waza `/think`, or `repo-harness-plan` produces a decision-complete plan, capture it with `scripts/capture-plan.sh --slug <slug> --title <title>` so `plans/` becomes the file-backed source of truth; if the user has already approved implementation, capture with `--status Approved --execute` or run `scripts/plan-to-todo.sh --plan <active-plan>`.
- If current repo state conflicts with the task, open an isolated `codex/<task-slug>` worktree, finish there, run Waza `/check`-style validation, then merge back to `main` without absorbing unrelated dirty changes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ancienttwo/repo-harness](https://github.com/Ancienttwo/repo-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
