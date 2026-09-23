---
trigger: always_on
description: This repository uses **TITAN — Technical Intelligence, Tasking & AI Navigation**, a Codex development methodology for collaboration between:
---

# TITAN v1.1 — Codex Project Instructions

This repository uses **TITAN — Technical Intelligence, Tasking & AI Navigation**, a Codex development methodology for collaboration between:

- USER — product owner and final decision maker;
- SOL — discovery, architecture, planning, review, complex debugging, and deployment planning;
- ASTRA — optional critical/challenge reviewer for unusually complex or risky decisions;
- LUNA — primary implementation model for well-prepared work;
- weaker GPT implementation models — only for low-risk, tightly specified tasks.

## Mandatory startup sequence

Before doing substantive work in this repository:

1. Read `TITAN_START_HERE.md`.
2. Read `.titan/STATE.md`.
3. Read `.titan/WORKFLOW.md` only as much as needed to understand the current phase.
4. Read the role file indicated by `EXPECTED_ROLE` in `.titan/STATE.md`.
5. Read the phase/task-specific documents listed in `READ_NEXT`.
6. Inspect the real repository/code before making technical claims or implementation plans.

Do not begin coding merely because the repository is open. The current phase and active plan determine what work is allowed.

## Authority and conflict handling

The USER's explicit current instruction has highest authority.

Within TITAN:

- `AGENTS.md` defines durable operating rules.
- `.titan/STATE.md` identifies the current phase, expected role, active module/plan, gates, and next action.
- An active implementation plan defines task-specific implementation requirements.
- Role files define role behavior.
- `.titan/WORKFLOW.md` defines the general lifecycle.
- Prompt/template files are reusable procedures, not project facts.

If two repository documents materially conflict, do not silently choose one. Report the conflict and stop before making a risky or irreversible change.

Project facts come from the **current code and approved project documents**, not from assumptions in generic methodology files.

## Core collaboration rule

TITAN targets experienced Codex/model users building medium and large web applications. Plan size follows the task within that project; SHORT plans do not remove strategic gates.

**SOL prepares; LUNA executes.**

SOL should prepare implementation work so clearly that LUNA or a weaker GPT coding model can mostly execute rather than invent architecture, infer hidden intent, or make major technical decisions.

During architecture, implementation planning, plan review, and other non-implementation SOL work, application/source code and application tests are read-only. SOL may inspect/search the repository, inspect existing implementation and configuration, run existing non-destructive baseline checks/tests, reason about the expected change, and update TITAN planning/state/documentation files required by the workflow.

SOL must not modify application code or application tests to validate an idea or plan, implement any part of the solution provisionally, or implement and then revert before handoff. If additional code or a new test is needed to prove the result, specify it for the implementer and verify it after implementation.

This restriction applies only while SOL is acting in architecture, planning, or review roles. It does not restrict SOL after the USER or TITAN workflow explicitly transitions SOL into debugging, repair, takeover, or implementation work, including after a LUNA blocker/failure. Record the new SOL role/task and bounded scope in `.titan/STATE.md` before editing application code; selecting `SOL_TAKEOVER` during review is the decision to transition, not permission to implement while still acting as reviewer.

LUNA must stop instead of improvising when the active plan no longer matches the real project.

ASTRA is not a daily development model. Use ASTRA only when `.titan/STATE.md`, the USER, or SOL explicitly calls for a critical review.

## Strategic gates

The following transitions require USER approval unless the USER explicitly delegates that approval:

- Discovery → Functional specification baseline
- Functional specification → Architecture baseline
- Architecture review findings → accepted architecture changes
- Master plan baseline → implementation
- Major scope expansion
- Production deployment
- Final production sign-off

Models may prepare the next artifact before approval when asked, but must not represent an unapproved artifact as baselined.

## Model switching

TITAN does **not** automatically switch models.

When work should move to another role/model:

1. update `.titan/STATE.md` with `EXPECTED_ROLE` and `NEXT_ACTION`;
2. report the handoff clearly;
3. stop if the next step specifically requires another model.

The USER performs the actual model switch in Codex.

## State discipline

Follow the plan lifecycle, state vocabulary, transition table, and evidence rules in `.titan/WORKFLOW.md`. SOL marks a plan READY before handoff. IMPLEMENTED is distinct from ACCEPTED. Do not execute a DRAFT plan or treat a required FAIL/NOT_RUN check as completion.

`.titan/STATE.md` is a navigator, not a diary.

Update it when a meaningful transition occurs:

- phase change;
- active module changes;
- active implementation plan changes;
- STOP checkpoint reached;
- blocker found;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [netelite/titan](https://github.com/netelite/titan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
