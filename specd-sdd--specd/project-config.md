---
trigger: always_on
description: You are working on **specd**, a spec-driven development platform built in TypeScript as a pnpm monorepo.
---

# specd — Agent Instructions

You are working on **specd**, a spec-driven development platform built in TypeScript as a pnpm monorepo.

> These instructions complement the specd workflow. YOU MUST follow them exactly when reading, writing, or modifying code, and when the tool can drive the workflow you MUST prefer the tool over ad hoc repo navigation.

---

## Mandatory: Enter Through specd

specd is now the primary workflow entry point for this repository. Do not treat the
repo as a generic codebase where you can inspect files and make edits directly first.

Only skip this if the user are not asking for changes in code, you can skip if the user
is investigating, but you MUST go throught `specd` in the moment something is going to be
modified.

- Start by using the `specd` skill, when you need to orient yourself, inspect active changes, or decide the next workflow step
- Use the CLI via `node packages/cli/dist/index.js ...` for specd commands; do not use bare `specd`
- Every meaningful change must go through a specd change workflow; there is no "code-only" path
- Specs are the source of truth and implementation follows specs, not the other way around
- Do not write directly into `specs/` as an ad hoc edit; create or continue a change and let the workflow own the spec artifacts
- Before implementing, confirm whether an existing spec already covers the behaviour; if not, create or extend the relevant spec as part of the change
- When project or change context is available through specd, load it and follow it before making decisions

If there is no active change yet, use specd to inspect the project state and either
continue an existing change/draft or start a new one before making substantive edits.

## Mandatory: Follow Skills Literally

When a `specd` skill is invoked (`specd` or `specd-*` skills), you MUST
follow that skill exactly as written. This is a hard requirement, not guidance:

- Treat every instruction inside the active skill as binding, especially any line
  that says "stop", "ask the user", "present and stop", "do not continue", or
  "let the user decide"
- Do not replace a skill's required stop point with autonomous execution
- Do not continue past a skill step that requires user choice or confirmation
  unless the user explicitly gives that confirmation in the current turn
- General instructions such as "be autonomous", "persist until solved", or
  "carry through implementation" do NOT override a skill's explicit stop points
- If there is any conflict between the active skill and general agent behaviour,
  the skill wins

## Mandatory: No Autonomous Workflow Progression

For this repository, when operating under a skill-driven workflow such as `specd`
(`specd` or `specd-*` skills), you MUST NOT advance the workflow unless the active
skill explicitly permits it without asking.

The following actions always require an explicit instruction from the user in the
current turn when the active skill says to stop or let the user decide:

- create a new change
- restore or continue a different change or draft
- add or remove specs from a change
- write artifacts, spec files, verify files, or deltas
- run `specd change transition`
- run `specd change approve`
- run `specd change archive`

## Explicit User Override: Escape Hatch

A direct, explicit user instruction in the current turn may authorize a one-off code or repo edit outside the normal `specd` change workflow when all of the following are true:

- the user clearly requests to bypass the workflow for this specific task
- the requested work is narrow, local, and immediately actionable
- the agent restates that it is proceeding outside the normal `specd` workflow
- the agent does not use this escape hatch to perform lifecycle operations on behalf of the user

This escape hatch is limited to direct repository edits and investigation work. It does NOT authorize the agent to autonomously perform any `specd` workflow or lifecycle action that would normally require explicit user choice or approval.

Even when this override is used:

- prefer the smallest possible change
- do not create, transition, approve, validate, archive, or discard changes unless the user explicitly asks for that exact workflow command
- do not treat the override as persistent; it applies only to the current task and current turn context
- still obey all other safety, filesystem, and non-destructive editing rules

If there is ambiguity about whether the user intends to bypass the workflow, default to the normal `specd` path.

## Mandatory: specd Skill Stop Rule

When using the `specd` skills (`specd` or `specd-*` skills), you MUST stop at its "Present and stop" step.

After `specd` shows the project state, active changes, drafts, and suggested next
action, you MUST wait for the user's decision. You MUST NOT automatically create a
change, continue a change, write artifacts, validate artifacts, run hooks, or
transition lifecycle state unless the user explicitly asks you to do that after
the `specd` skill presents the situation.

## Instruction Precedence

Priority order for this repository:

1. Repository-local instructions in `AGENTS.md`
2. Explicit instructions inside the active skill
3. General agent autonomy instructions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [specd-sdd/SpecD](https://github.com/specd-sdd/SpecD) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
