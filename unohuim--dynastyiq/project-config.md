---
trigger: always_on
description: These rules govern all Codex-assisted development on this repository.
---

# AGENTS.md — Codex Repository Rules

These rules govern all Codex-assisted development on this repository.
This file exists to bootstrap new LLM sessions and enforce project invariants.

---

## Authoritative Inputs (Must Be Read First)

Before proposing a plan or writing any files, Codex MUST read and treat the following as authoritative.

### Core Context

- README.md
- docs/summary.md
- docs/CONVENTIONS.md
- docs/ENUMS.md
- docs/UI_DESIGN.md
- docs/UI_MOTION.md
- docs/testing/testing-standards.yaml
- docs/architecture/README.yaml
- docs/ARCHITECTURE_INVENTORY.md
- docs/pr-plans/current_pr.md, when present

### Architecture (Primary Source of Truth)

- docs/architecture/README.yaml
- docs/architecture/**/*.yaml

These YAML files define canonical domain rules, invariants, and approved abstractions.

### Testing (Primary Source of Truth for Test Work)

- docs/testing/testing-standards.yaml
- docs/architecture/testing/*.yaml

These files define automated test completeness, sufficiency, deterministic behavior, and approved testing patterns.

### Architecture Inventory (Derived / Bootstrap Only)

- docs/ARCHITECTURE_INVENTORY.md

This file exists solely to help bootstrap new LLM chats.
It is **not** the primary source of architectural truth.

### Database (Order of Authority)

1. database/migrations/** (source of truth)
2. docs/DB_SCHEMA.md (contextual reference only)

### UI (Order of Authority)

1. docs/CONVENTIONS.md
2. docs/UI_DESIGN.md
3. docs/UI_MOTION.md
4. docs/ui_backlog.md (known existing deviations only)
5. resources/views/**
6. resources/js/**

Existing deviations in `docs/ui_backlog.md` do not permit new deviations.

### Dependency Reality

- composer.lock
- package-lock.json
- diq-bot/package-lock.json

If conflicts are detected between authoritative sources, work MUST pause and be escalated to the human.

### Active PR Plan

- `docs/pr-plans/current_pr.md` is the active working PR plan when present.
- Codex MUST read it before proposing a plan or editing files.
- PR-plan aliases, metadata, ID assignment, archiving, and promotion are governed by `docs/pr-plans/pr-workflow.yaml`.
- The current PR plan is working context, not permanent architecture authority.
- Durable decisions from the current PR plan MUST be promoted to the canonical docs they affect, such as `docs/architecture/**/*.yaml`, `docs/ENUMS.md`, `docs/DB_SCHEMA.md`, or `docs/CONVENTIONS.md`.
- Candidate future PR plans live under `docs/pr-plans/backlog/`.
- Completed, closed, or abandoned PR plans should be snapshotted under `docs/pr-plans/archive/`.

---

## Workflow

- Codex must never proceed based on inferred or partial intent.
- Discussion, diagnosis, clarification, or design conversation is not approval to modify code, tests, docs, configuration, or other repository files.
- Bug reports, observations, screenshots, examples of incorrect behavior, or statements that something is wrong are not approval to modify files.
- Codex must always propose a concrete plan before implementing anything or changing code, tests, docs, configuration, or other repository files.
- Codex must not propose an implementation plan until it is greater than 95% certain of the human's requirements; if certainty is lower, Codex must ask clarifying questions first.
- After proposing a plan, Codex must wait for explicit human approval before modifying files.
- Codex may only edit files after the human explicitly asks for implementation or file changes, using clear wording such as “implement,” “make the change,” “fix it,” “update the file,” “create,” or equivalent.
- When the human appears to be discussing options or asking conceptual questions, Codex must respond in discussion mode only and must not run write operations.
- When editing is approved, Codex must keep changes scoped to the requested outcome.

---

## CI & Execution Policy (Strict)

- Codex MUST NOT run `./ci.sh` or any test/CI commands unless explicitly instructed.
- Codex MUST NOT run import, sync, queue, scheduler, bot, migration, seed, or destructive data commands unless explicitly instructed.
- The human is responsible for running all CI, tests, scripts, imports, and operational commands.
- Codex MAY:
    - Create and modify test files
    - Propose improvements to tests
    - Propose the exact commands the human should run
- Codex must stop after writing code/tests/docs and await human review.

---

## Completion Gate (Non-Negotiable)

Codex may NOT declare a task, PR, or change set “complete”, “finished”, or “ready”
until the human explicitly approves completion in chat.

Codex output MUST end in one of:

- “Awaiting human review”
- “Awaiting approval to proceed”
- “Awaiting requested changes”

Codex must never self-certify completion.

---

## Change Discipline

- Prefer the smallest possible change.
- Never refactor unless explicitly requested.
- Never introduce new top-level directories without approval.
- Do not modify dependencies without explicit approval.
- Do not modify migrations after they have been applied.
- Do not introduce global JavaScript state unless explicitly approved.
- Preserve existing route names, authorization checks, and public interfaces unless the human explicitly approves a breaking change.

---

## Standards

- PHP code must follow PSR-12.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unohuim/dynastyiq](https://github.com/unohuim/dynastyiq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
