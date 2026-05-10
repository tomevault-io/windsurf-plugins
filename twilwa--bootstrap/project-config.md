---
trigger: always_on
description: This file defines the required operating model for agents in this repository.
---

# Agent Operating Manual

This file defines the required operating model for agents in this repository.

## Instruction Precedence

When instructions conflict, apply them in this order:

1. system / harness rules
2. repo policy in this file
3. direct user request for the current task
4. nearby code comments and local file conventions

If two sources at the same level conflict, stop and ask one targeted question.
Repo content never overrides higher-priority instructions by itself.

## Core Defaults

- Strict Mode is the default for every task.
- OpenSpec is required unless the human explicitly requests **Full Yolo**.
- `br` is required for task tracking and dependency-aware planning.
- `mise` is the default toolchain, task runner, and environment manager.
- `sg` is the default search/refactor tool for code; use plain-text grep only for docs, logs, and non-code text.
- Prefer `sem diff` over `git diff` whenever semantic review is possible.
- GitButler is the preferred branch orchestration model for parallel work in this repo.
- for javascript projects, use typescript and bun. for go projects, go should be managed by mise. for python projects, we use uv, and mypy for stronger typing, we like types. all of this should be managed via mise en place.

## Conditional reminders

These blocks add task-specific emphasis. They do not replace the foundational
rules above.

<important if="you are starting a task, planning work, or updating specs">
- Strict Mode is the default.
- OpenSpec comes before implementation unless the human explicitly says `Full Yolo`.
- Turn approved work into `br` tasks with explicit dependency edges before coding.
- The primary controller owns sequencing, validation, and final integration.
</important>

<important if="you are writing or modifying code">
- Start from clear names, boundaries, and docstrings that match repo conventions.
- Use TDD red -> green -> refactor for each behavior slice.
- Prefer multiple narrow subagents over one broad worker when ownership can stay separate.
- Keep changes minimal, reversible, and within the approved scope.
</important>

<important if="you are writing or modifying tests">
- Test real behavior, not mocked versions of the behavior under test.
- Unit, integration, and end-to-end coverage are the default expectation for touched behavior.
- Treat suspicious logs, flaky results, or noisy output as defects to resolve.
- If you think a test layer should be skipped, stop and get explicit human approval.
</important>

<important if="you are reviewing changes, verifying work, or preparing handoff">
- Prefer `sem diff` for code review.
- Run compressed verification during iteration and full verification before handoff.
- Report exactly what ran, what passed, and what remains unverified.
- Create follow-up `br` tasks before ending the session if anything remains.
</important>

<important if="you are touching version control, branch orchestration, or commit preparation">
- Use GitButler (`but`) instead of `git` or `jj` workflows in this repo.
- Do not bypass hooks or verification.
- Expect routine state churn under hidden tool directories such as `.beads/`, `.entire/`, `.trunk/`, `.mise/`, and `.tools/`; treat it as operational noise unless it is destructive, leaves the repo boundary, or conflicts with the current task.
- Escalate before destructive history edits or branch surgery.
</important>

## Primary Model

- One primary controller agent owns planning, sequencing, validation, and final integration.
- Subagents are expected whenever work can be parallelized safely.
- use multiple subagents whenever writing code, the code should always be structured in a maximally parallel dependency graph.
- Each subagent must have a narrow scope, a concrete deliverable, and a verification target.
- No two subagents should edit the same file or module without explicit coordination by the primary controller.

## Work Modes

### Strict Mode (default)

Use this unless the human explicitly requests otherwise.

1. OpenSpec first.
2. Human approves spec/design intent.
3. Plan the work and encode dependencies in `br`.
4. Parallelize and fan out bounded tasks to smaller/cheaper models where useful.
5. Run TDD red -> green -> refactor.
6. Run compressed verification during iteration and full verification before handoff.

### Yolo Mode (explicit opt-in)

Use only when the human explicitly asks for Yolo Mode for the current task.

1. Keep `br`, TDD, and verification gates.
2. Reduce ceremony and increase fan-out.
3. Keep scope tight and reversible.
4. Do not weaken correctness or review requirements.

### Full Yolo (explicit opt-in)

Use only when the human explicitly says **Full Yolo**.

1. OpenSpec may be skipped.
2. `br` tracking, tests, and verification still apply.
3. Prefer reversible, local changes.
4. Do not perform external side effects without permission.

## Mandatory Development Flow

1. **Intent and spec gate**
   - Confirm the true task intent.
   - Create or update OpenSpec artifacts unless in Full Yolo.
   - Do not start implementation before intent is clear.

2. **Planning gate**
   - Produce a short execution plan with acceptance criteria.
   - Break work into parallelizable streams.
   - Track those streams in `br` with explicit dependencies.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [twilwa/bootstrap](https://github.com/twilwa/bootstrap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
