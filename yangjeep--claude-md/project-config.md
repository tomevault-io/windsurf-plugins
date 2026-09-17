---
trigger: always_on
description: This file is the repository-level execution contract for coding agents.
---

# CLAUDE.md — Project Agent Contract

This file is the repository-level execution contract for coding agents.

Keep it small and high-signal. It should contain durable constraints and recurring traps, not a description of the entire codebase.

When this template is used for another repository, customize the **Project Contract** section and keep the operating rules unless the project has a deliberate reason to change them.

## Project Contract

Fill these in for the repository.

- **Mission:** `<one or two sentences describing the product or system outcome>`
- **Authoritative architecture:** `<ADRs, architecture docs, or repository paths that define durable constraints>`
- **Task authority:** `<GitHub issues, tracker, or other source that defines current scope and acceptance criteria>`
- **Primary validation:** `<commands that must pass before a normal change is complete>`
- **Architecture decisions:** `.agent/ADR/`
- **Worklog:** `.agent/WORKLOG/`
- **Hard invariants:**
  - `<durable, non-obvious rule that would be expensive to violate>`
  - `<durable, non-obvious rule that would be expensive to violate>`
  - `<durable, non-obvious rule that would be expensive to violate>`

Do not turn this section into an architecture manual. If an agent can discover a fact cheaply from code, tests, generated schemas, or configuration, prefer repository inspection over duplicating it here.

## 1. Read and Investigate First

Before non-trivial implementation:

1. Read this file.
2. Read the issue, task, or explicit user instruction that authorizes the work.
3. Inspect the relevant code, tests, configuration, and recent repository history.
4. Read referenced product, architecture, ADR, or worklog material only when it is relevant to the task.
5. Establish a green baseline with the narrowest relevant existing tests when practical.

Repository reality is authoritative for what exists today. Do not speculate about code you have not inspected when the repository can answer the question.

Do not assume a test, lint, build, or CI failure is pre-existing. Verify the baseline before using that explanation.

If repository reality materially conflicts with the task contract or an architectural invariant, do not silently choose one. Surface the conflict when resolving it would change product behavior, architecture, security, data integrity, or migration risk.

## 2. Execution, Not Plan Theater

When the user or task has already authorized implementation, execute the work end-to-end.

A plan is a working artifact, not a stopping point. Do not ask for another approval merely to:

- begin implementation after planning;
- choose between ordinary equivalent implementation details;
- inspect source, tests, configuration, or documentation;
- run normal tests, linters, type checks, builds, or local verification;
- fix ordinary implementation, test, lint, or CI failures;
- make routine repository-local decisions supported by existing patterns.

Proceed autonomously through the authorized scope and dependency order.

Ask for human input only when there is a genuine blocker involving:

- unresolved product behavior;
- architecture outside established constraints;
- meaningful migration or data-loss risk;
- security-risk acceptance;
- missing external credentials or provider capability requiring human action;
- an irreversible or destructive production action;
- two materially different valid outcomes that cannot be resolved from repository evidence.

## 3. Smallest Working Change

Default to the smallest coherent change that satisfies the task.

- Follow existing repository patterns before inventing new ones.
- Do not bundle unrelated cleanup, formatting, or refactoring.
- Do not add speculative abstractions, compatibility layers, configurability, or infrastructure.
- Do not add a dependency when existing platform or repository capabilities are sufficient.
- Preserve public contracts and backward compatibility unless the task explicitly changes them.
- Put follow-up ideas outside the current change rather than expanding scope.

One focused problem per issue and PR is the default. Large work should be decomposed into independently verifiable changes when the repository workflow allows it.

## 4. Deterministic Boundaries

Use model judgment for interpretation, synthesis, classification, and other work where judgment is the intended behavior.

Use deterministic mechanisms for hard guarantees such as:

- authorization and tenant/account boundaries;
- schema and contract validation;
- idempotency, deduplication, and replay safety;
- migration and rollback behavior;
- destination and external-write authorization;
- budget, rate, or safety ceilings;
- invariants that must hold regardless of model behavior.

AI output is never authoritative state merely because a model produced it. If the product uses AI in a business workflow, the repository must define how proposals become validated state.

## 5. Testing and Verification

Never claim something was tested unless the relevant command actually ran successfully.

For bug fixes and deterministic behavior changes, prefer a RED-first loop:

1. Reproduce the defect or missing behavior with a focused failing test.
2. Confirm the test fails for the expected reason.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yangjeep/CLAUDE.md](https://github.com/yangjeep/CLAUDE.md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
