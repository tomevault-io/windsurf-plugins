---
trigger: always_on
description: Use this skill for early-stage software projects that need phase-based planning, fixtures-first and tests-first implementation, scope control, change request tracking, verification, and handoff documents for continuing work across Codex sessions.
---


# phase-workflow

Use this skill to keep early-stage project work phase-based, testable, and recoverable across
Codex sessions.

## Mandatory Invocation

If `phase-workflow` applies, first open this `SKILL.md` in the current turn.
Then read only the needed reference file for the current decision.

Hook-injected reminders do not count as skill invocation. Project `AGENTS.md` rules do not
replace opening this skill when the skill applies. Do not rely on compressed chat history or
remembered `phase-workflow` rules.

Before mutating files, state the active authorization branch: plan-change, phase start gate,
post-gate execution, approach confirmation, status/handoff update, or recovery-record
mutation.

## Resource Navigation

Load only the resource needed for the current decision:

- Read `references/phase_policy.md` when starting, exiting, renumbering, or splitting a phase.
- Read `references/change_request_policy.md` when a new request appears during an active phase.
- Read `references/handoff_protocol.md` when resuming work in a new window or ending a work round.
- Read `references/verification_policy.md` before marking a phase complete or reporting test status.
- Copy from `templates/` when creating phase notes, handoff notes, TODOs, or decision records
  in a target project.
- Copy `templates/project_context_template.md` as `PROJECT_CONTEXT.md` when adopting an
  existing structured project.
- Use `examples/` only when the user needs a concrete pattern for a greenfield start, existing
  structured project adoption, change request, or new-window handoff.

## When To Use This Skill

Use this skill when the user is working on:

- A new project.
- An existing project with a clear structure that needs lightweight phase planning and handoff
  files.
- An MVP project.
- A Phase 0 initialization.
- A phase-based development plan.
- Scope control for a small or changing project.
- New-window handoff after prior Codex work.
- Mid-phase change request handling.
- Updates to `TODO.md`, `DECISIONS.md`, phase notes, or handoff documents.

## When Not To Use This Skill

Do not use this skill when:

- The user is only asking a simple question.
- The user explicitly asks for a direct small bug fix in a project that has not adopted
  `phase-workflow`, or explicitly opts out of `phase-workflow` for that request.
- The project is mature and already has a process the user does not want to change.
- The project state is unclear, heavily tangled, or lacks an identifiable entrypoint,
  verification command, or current goal.
- The current task does not need long-term context management.
- The repository has higher-priority development instructions.

In a project that has adopted `phase-workflow`, a direct small bug fix is still governed by
phase-workflow. Classify it as Phase X.2 unless the user explicitly opts out of phase-workflow
for that request. An opt-out does not rewrite project workflow records; it only scopes that
request outside the workflow.

## Core Workflow

For each phase:

1. Lock phase scope.
2. Prepare fixtures/examples.
3. Write failing tests.
4. Implement the minimal capability.
5. Verify with actual commands.
6. Update `TODO.md`, phase notes, and handoff/current-state.
7. Move to the next phase only after verification.

## Authorization Model

Use this single authorization model before applying the detailed rules below. When two signals
conflict, use the narrower authorization and stop at the next required gate.

| Signal | Authorizes | Does not authorize | Required stop |
| --- | --- | --- | --- |
| Plan-change confirmation | planning-file updates only | Tests, code, scripts, migrations, restores, or other technical files | Stop after planning files |
| Phase start request | phase analysis and visible start gate only | Execution, file edits, or current-phase mutations | Stop after the gate |
| Post-gate execution confirmation | current-phase technical mutations that pass mutation preflight | Next-phase work, multi-phase execution, planning-file mutations, recovery-record mutations, or unstated approach choices | Stop when the current phase exits |
| Approach confirmation | approved non-trivial approach inside the current phase | Phase boundary changes or broader scope | Stop if the approach is rejected |
| Mutation preflight pass | The checked current-phase mutation branch | Failed-gate, unconfirmed, wrong-branch, next-phase, or multi-phase mutations | Stop before files if any check fails |
| Phase violation detected | read-only recovery audit and chat-visible incident report before user choice; recovery record repairs after user choice | New implementation, next-feature work, continuing the violated flow, or pre-choice record repair | Stop before record repairs until the user chooses keep-and-audit or rollback |

This model keeps plan-change confirmation, phase start requests, post-gate execution
confirmation, approach confirmation, mutation preflight, and phase violation recovery as
separate controls. They cannot substitute for each other and do not authorize multi-phase

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FossilLeeSZ/phase-workflow](https://github.com/FossilLeeSZ/phase-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
