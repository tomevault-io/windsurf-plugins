---
trigger: always_on
description: FLOW is a Claude Code plugin (`flow:` namespace) that enforces an opinionated 6-phase development lifecycle: Start, Plan, Code, Code Review, Learn, Complete. Each phase is a skill that Claude reads and follows. Phase gates prevent skipping ahead — you must complete each phase before entering the next. Language-agnostic — every project owns its toolchain via repo-local `bin/format`, `bin/lint`, `bin/build`, `bin/test` scripts that FLOW orchestrates without dispatching by language.
---

# CLAUDE.md

FLOW is a Claude Code plugin (`flow:` namespace) that enforces an opinionated 6-phase development lifecycle: Start, Plan, Code, Code Review, Learn, Complete. Each phase is a skill that Claude reads and follows. Phase gates prevent skipping ahead — you must complete each phase before entering the next. Language-agnostic — every project owns its toolchain via repo-local `bin/format`, `bin/lint`, `bin/build`, `bin/test` scripts that FLOW orchestrates without dispatching by language.

This repo is the plugin source code. When installed in a target project, skills and hooks run in the target project's working directory, not here. State files, worktrees, and logs all live in the target project. If you are developing FLOW itself, you are modifying the plugin — not using it.

## Design Philosophy

Four core tenets guide every design decision:

1. **Unobtrusive** — zero dependencies. Prime commits `.claude/settings.json` and the four `bin/*` stubs as project config. `.flow.json` is always git-excluded. Everything else lives in `.git/` or is gitignored.
2. **As autonomous or manual as you want** — configurable autonomy via `.flow.json` skills settings.
3. **Safe for local env** — no containers needed, no permission prompts ever. Native tools only, no external dependencies.
4. **N×N×N concurrent** — N engineers running N flows on N boxes at the same time is the primary use case, not an edge case. Every feature, fix, and design decision must work when multiple flows are active simultaneously — on the same machine (multiple worktrees) and across machines (shared GitHub state). Local state (`.flow-states/`, worktrees) is per-machine. Shared state (PRs, issues, labels) is coordinated through GitHub. Nothing assumes a single active flow.

In the target project:

- `.claude/settings.json` and the `bin/*` stubs are committed during prime. `.flow.json` is always git-excluded
- `.flow-states/` is gitignored and deleted at Complete
- After Complete, the only permanent artifacts are the merged PR and any CLAUDE.md learnings
- Skills are pure Markdown instructions, not executable code
- Tool dispatch is repo-local: `bin/flow ci` runs `./bin/format`, `./bin/lint`, `./bin/build`, `./bin/test` from cwd. Each repo owns its commands; FLOW provides the orchestration layer (sentinel, retry/flaky classification, recursion guard, fail-fast ordering, JSON contract). Adding a language means writing the four `bin/*` scripts in your project, not editing FLOW
- Multiple flows run simultaneously via branch-scoped worktrees and state files — nothing assumes a single active flow

## The 6 Phases

| Phase | Name | Command | Purpose |
|-------|------|---------|---------|
| 1 | Start | `/flow:flow-start` | Create worktree, PR, state file, configure workspace |
| 2 | Plan | `/flow:flow-plan` | Invoke decompose plugin for DAG analysis, explore codebase, create implementation plan |
| 3 | Code | `/flow:flow-code` | Execute plan tasks one at a time with TDD |
| 4 | Code Review | `/flow:flow-code-review` | Six tenants assessed by four cognitively isolated agents (reviewer, pre-mortem, adversarial, documentation) launched in parallel. Parent gathers, triages, and fixes. |
| 5 | Learn | `/flow:flow-learn` | Review mistakes, capture learnings, route to permanent homes |
| 6 | Complete | `/flow:flow-complete` | Merge PR, remove worktree, delete state file |

Phase gates are enforced by `bin/flow check-phase` (`src/check_phase.rs`) — there is no instruction path to skip a phase. Back-transitions (e.g., Code Review can return to Code or Plan) are defined in `flow-phases.json`.

## When You Must Update Docs and Tests

"Marketing docs" refers to `docs/index.html` — the GitHub Pages landing page.

### Structural sync (CI-enforced by `tests/docs_sync.rs`)

CI will fail if these are missing:

- New/renamed skill — `docs/skills/<name>.md`, `docs/skills/index.md`, `README.md`
- New/renamed phase — `docs/phases/phase-<N>-<name>.md`, `docs/skills/index.md`, `README.md`, `docs/index.html`
- New feature/capability — `README.md` and `docs/index.html` must mention required keywords (see `required_features()` in `tests/docs_sync.rs`)

### Content sync (convention-enforced — no test catches this)

- Changed skill behavior (new flag, changed steps, different workflow) — update `docs/skills/<name>.md` and the Description column in `docs/skills/index.md` to match
- Changed phase behavior — update `docs/phases/phase-<N>-<name>.md` and the Description column in `docs/skills/index.md` to match
- Changed architecture or capabilities — update `README.md` and `docs/index.html` if the change affects how FLOW is described to users

### Test requirements

- New skills auto-covered by `tests/skill_contracts.rs` (glob-based discovery)
- Any new executable code needs tests — skills are Markdown and don't need tests beyond contracts

## Key Files


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [benkruger/flow](https://github.com/benkruger/flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
