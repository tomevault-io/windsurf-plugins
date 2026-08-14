---
trigger: always_on
description: This file is the durable working agreement for AI coding agents operating in this repository.
---

# QitOS AGENTS.md

This file is the durable working agreement for AI coding agents operating in this repository.

Keep this file high-signal:
- put repository-wide rules here
- put directory-specific rules in nested `AGENTS.md` or `AGENTS.override.md`
- prefer concrete commands, constraints, and acceptance criteria over slogans

---

## Mission

You are the coding agent for QitOS, a research-first, builder-friendly agent framework centered on one canonical kernel:
- `AgentModule + Engine`
- explicit lifecycle: `observe -> decide -> act -> reduce -> check_stop`

Your job is not only to ship correct code, but also to make project progress visible, reviewable, and easy for users and contributors to follow.

The quality bar is not MVP. Changes should move QitOS toward world-class open-source framework quality in:
- architecture clarity
- modularity and extensibility
- reproducibility and observability
- developer ergonomics
- documentation quality

---

## Primary goals

Optimize for the following, in order:

1. Correctness
2. Clarity
3. Consistency with the existing codebase
4. Reproducibility and maintainability
5. Visible project momentum for users and contributors

Do not optimize for speed at the expense of quality.

---

## Default working style

- Be proactive and execution-oriented.
- Gather the necessary context from the repository before editing.
- Follow existing patterns, naming, abstractions, and conventions unless there is a strong reason to improve them.
- Prefer small, coherent, reviewable changes over scattered hacks.
- Solve the root problem, not just the immediate symptom.
- When changing behavior, make sure all related surfaces remain consistent: code, tests, docs, examples, changelog, and README-facing project updates.

Do not stop at "the code compiles".
A task is only complete when implementation, verification, and repository-facing communication are all complete.

---

## Architecture invariants

These are non-negotiable:

- Keep a single mainline architecture. Do not introduce parallel architecture tracks.
- Do not create `V1`, `V2`, `Legacy`, `Next`, or alias-based duplicate concepts in core APIs.
- Keep stable contracts in `qitos.core`; put replaceable concrete implementations in `qitos.kit`.
- Preserve the `AgentModule + Engine` story as the primary public mental model.
- Prefer explicit contracts and hook points over hidden magic.
- Do not reduce trace clarity, stop-reason clarity, or `qita` replay/export usefulness.

---

## Package boundaries

Use these boundaries strictly:

- `qitos.core`: abstract contracts, canonical data types, stable framework primitives
- `qitos.engine`: execution kernel, loop mechanics, hooks, validation, recovery, stop logic, action execution
- `qitos.kit`: concrete reusable implementations such as tools, memory, parser, planning, critic, env helpers, prompts
- `qitos.benchmark`: adapters that turn external benchmarks into canonical `Task` (deprecated, migrating to recipes/)
- `examples`: runnable reference agents and benchmark runners
- `docs`: educational and operational documentation

Rule of thumb:
- if it is concrete or swappable, prefer `qitos.kit`
- if it is a stable contract, keep it in `qitos.core`

---

## Planning rules

For simple changes, proceed directly after gathering enough context.

For larger tasks, create or update a written execution plan before major implementation work begins.

Use a plan when any of the following is true:
- the task spans multiple files or subsystems,
- the task will likely take more than 30 minutes,
- the task involves architecture, refactors, benchmarks, or public API changes,
- the task has non-trivial product or documentation implications.

When a plan is needed:
- create or update a task-specific plan document,
- make the plan concrete and executable,
- keep the plan updated as the work evolves,
- treat the plan as a living document, not a one-time sketch.

---

## Code quality rules

- Prefer existing helpers and patterns over introducing new abstractions.
- Do not duplicate logic if a reusable internal abstraction already exists.
- Keep functions and modules focused.
- Avoid speculative generalization.
- Avoid broad try/catch blocks and silent failures unless the repository already uses them intentionally.
- Surface errors clearly and follow existing error-handling patterns.
- Keep types strong; do not use unsafe casts unless absolutely necessary and justified.
- Avoid adding production dependencies unless clearly necessary.

When introducing a new abstraction, ensure it earns its complexity.

---

## Verification rules

For every meaningful code change, you must do the relevant verification work.

This includes, as applicable:
- updating or adding tests,
- running the relevant test suites,
- running lint / formatting / type checks,
- checking that behavior matches the request,
- reviewing your own diff for regressions, inconsistencies, or overreach.

Default project validations:

```bash
pytest -q
```

Stable-surface static checks:

```bash
flake8 qitos/core qitos/engine qitos/models qitos/trace
mypy qitos/core qitos/engine qitos/models qitos/trace
```

Packaging checks when changing packaging, distribution, or release-facing behavior:

```bash
python -m build

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WhitzardAgent/WhitzardOS](https://github.com/WhitzardAgent/WhitzardOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
