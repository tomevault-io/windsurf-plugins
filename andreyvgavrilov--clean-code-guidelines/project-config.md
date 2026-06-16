---
trigger: always_on
description: Use this workflow for feature implementation requests. The goal is to separate research, planning, approval, and implementation so the final work is grounded in the codebase, explicitly scoped, and approved by the user before code changes begin.
---

# Agent Feature Implementation Workflow

Use this workflow for feature implementation requests. The goal is to separate research, planning, approval, and implementation so the final work is grounded in the codebase, explicitly scoped, and approved by the user before code changes begin.

## Principles

### Think Before Coding

Do not assume silently. Surface uncertainty before writing code.

- State assumptions explicitly.
- If the request has multiple reasonable interpretations, present them instead of choosing silently.
- Push back when a simpler approach would satisfy the goal.
- Stop when confused. Name what is unclear and ask for clarification.

Subagents cannot ask the user directly. When a subagent needs human input, it must stop and return a human input request to the orchestrator.

### Simplicity First

Use the minimum code that solves the approved goal.

- Do not add features beyond what was requested.
- Do not create abstractions for single-use code.
- Do not add flexibility, configurability, or extension points unless requested.
- Do not add error handling for impossible scenarios.
- If the implementation can be reduced substantially without losing behavior, simplify it before finishing.

### Surgical Changes

Touch only what is necessary.

- Do not improve adjacent code, comments, names, formatting, or structure while passing through.
- Do not refactor unrelated code.
- Match the existing style, even if a different style would be preferred.
- Remove imports, variables, functions, tests, and files that became unused because of this change.
- If unrelated dead code or cleanup is noticed, mention it in the final result instead of deleting it.

### Goal-Driven Execution

Turn every implementation request into a verifiable goal.

- Define success criteria before implementation.
- Never implement a feature before writing a failing test that verifies the goal.
- After writing code, if the test still fails, assume the problem is in the code by default.
- Change tests only when necessary, and only if the revised tests still verify the original goal.
- Continue until the success criteria are verified or a real blocker is surfaced.

## Orchestrator and Subagent Communication

The main agent is the orchestrator. Subagents do focused work, but the orchestrator owns all communication with the user.

If a subagent encounters ambiguity, confusion, a meaningful tradeoff, a simpler alternative that changes scope, or a decision that requires product judgment, it must pause and return a human input request instead of guessing.

A human input request should include:

- The unclear point or decision.
- Why it matters.
- The reasonable options, if known.
- The subagent's recommended option, if it has one.

When the orchestrator receives a human input request, it must ask the user for clarification or approval. After the user responds, the orchestrator must return the user's feedback to the subagent as additional input and let the subagent continue from that point.

If the user's answer changes the feature substantially, the orchestrator should route back to research or planning as described in Step 3.

## Step 1: Research

Call a subagent to research the information needed for implementation.

The research subagent should inspect the relevant code, tests, documentation, configuration, and existing patterns. It should not implement the feature. It should return a concise research result that includes:

- Relevant files, modules, and ownership boundaries.
- Current behavior and data flow.
- Existing patterns that should be followed.
- Risks, unknowns, constraints, and likely test surfaces.
- Ambiguities, assumptions, and multiple possible interpretations.
- Simpler alternatives or reasons to push back, if any.
- Decisions that need to be made before implementation.

If the research subagent cannot produce a useful research result without user input, it must return a human input request to the orchestrator. The orchestrator must ask the user and return the user's answer to the research subagent before research continues.

The main agent should preserve the research result and use it as input for the planning step.

## Step 2: Planning

Call a clean subagent to create the feature plan.

Input to the planning subagent:

- The user's original request.
- The research result from Step 1.

The planning subagent should not implement the feature. It should return an implementation plan that includes:

- Success criteria.
- Proposed approach.
- Files expected to change.
- Data model, API, UI, or behavior changes, if applicable.
- Failing test or tests to write before implementation.
- Focused verification commands to run after implementation.
- Rollout or migration notes, if applicable.
- Open questions or assumptions.

If the planning subagent needs product judgment, scope clarification, or approval of a tradeoff before it can create a responsible plan, it must return a human input request to the orchestrator. The orchestrator must ask the user and return the user's answer to the planning subagent before planning continues.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andreyvgavrilov/clean-code-guidelines](https://github.com/andreyvgavrilov/clean-code-guidelines) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
