---
trigger: always_on
description: This file is the first stop for AI agents working in this repository.
---

# Agent Guide

This file is the first stop for AI agents working in this repository.

## Project

Task Monki is a local task board for running AI coding work in isolated Git
worktrees. It delegates implementation to first-class agent runtimes while Task
Monki keeps independent evidence for Git, tests, GitHub delivery, workflow
state, and local acceptance.

## Read Before Editing

- `docs/README.md`
  - Documentation map and docs policy.
- `docs/PRODUCT_WORKFLOW.md`
  - Product phases, action rules, and UI priorities.
- `docs/APP_SERVER_ARCHITECTURE.md`
  - Codex-specific App Server process, protocol, and recovery rules.
- `docs/architecture/AGENT_RUNTIME_ARCHITECTURE.md`
  - Runtime registry, identity, provider capability, routing, and recovery
    rules shared across integrations.
- `DESIGN.md`
  - Frontend and design guidance for coherent Task Monki UI changes.
- `docs/workflows/AGENT_REVIEW_WORKFLOW_LIFECYCLE.md`
  - Required reading before changing review, request-changes, stale-review,
    follow-up, or interrupt behavior.
- `docs/workflows/PR_STATUS_CARD_FLOW.md`
  - Required reading before changing PR Status, GitHub delivery evidence, or
    merge/check completion rules.
- `docs/architecture/CODEX_PROTOCOL_AND_COUPLING_NOTES.md`
  - Required reading before changing Codex protocol handling or generated
    bindings.

## Core Invariants

- Task Monki is authoritative for tasks, workflow phase, worktrees, Git state,
  test state, GitHub delivery state, and acceptance.
- Each agent runtime is authoritative only for its own processes, sessions,
  turns, items, approvals, plans, settings, models, and usage events.
- Provider output is telemetry, not verified evidence.
- Local Git/test/GitHub checks must be observed by Task Monki before they affect
  workflow or delivery decisions.
- An agent review is a detached quality gate inside the Review phase.
- Requesting review changes starts follow-up implementation work and belongs in
  In Progress until that work finishes.
- Stale review findings may be shown as context, but they must not be treated as
  current actionable verdicts.

## Common Commands

```sh
npm run typecheck
npm run check:architecture
npm test
npm run test:renderer:dom
npm run build
npm run check:codex-protocol
git diff --check
```

Run targeted tests while iterating, then run the full relevant set before
finishing changes that touch storage, workflow, protocol, or renderer behavior.

## Seeded UI And Workflow Testing

- Before testing UI or workflow states, run `npm run dev:seed` and use the
  generated `.local/task-monki-dev-seed/manifest.json`.
- Start local development from the generated environment:
  `source .local/task-monki-dev-seed/dev-api.env`, then `npm run dev:api` and
  `npm run dev:renderer`.
- Use stable scenario slugs such as `[seed:delivery-checks-failed]` instead of
  guessing app state or manually clicking through setup.
- If an important state is missing, extend `src/dev/seedData.ts` and
  `src/dev/seedData.test.ts`; do not rely on stale static fixtures or
  hand-edited store JSON.
- `scripts/serve-readme-screenshot-data.mjs` is screenshot-only legacy data and
  is not authoritative for workflow testing.

## Development Rules

- Keep edits scoped to the requested behavior.
- Before introducing a patch, understand why the issue happens. Then ask why it
  is happening now, why it did not happen before, and what changed in state,
  data, lifecycle, timing, or dependencies.
- Review nearby code and similar cases elsewhere in the repo before choosing an
  approach. Prefer existing patterns, state transitions, guards, and helper
  APIs over new one-off logic.
- Do not make the first solution "patch until it works." A fix should explain
  the underlying cause, preserve the product invariants, and avoid creating a
  second inconsistent path.
- Do not modify generated protocol files by hand.
- Regenerate protocol bindings only with `npm run generate:codex-protocol`.
- Do not mix protocol regeneration with product behavior changes.
- Keep provider-specific logic inside provider adapters and protocol mapping
  code.
- Keep UI workflow decisions based on Task Monki projections and verified
  evidence, not raw provider events.
- Update docs when behavior or invariants change.
- Do not commit, push, reset, clean, or discard changes unless the user
  explicitly asks. Treat unknown working-tree changes as user-owned.

## Where Code Belongs

- `src/core`
  - Domain logic, storage, projection, orchestration, provider adapters,
    process supervision, Git/test/GitHub services.
- `src/renderer/model`
  - Pure renderer selectors, derived UI state, formatting helpers, and
    testable view-model logic.
- `src/renderer/ui`
  - Presentation components and user interactions. Avoid putting workflow
    truth here.
- `src/shared`
  - Contracts and types shared by core and renderer. Changes here can affect
    stored data and IPC/API compatibility.
- `docs`
  - Current operational docs only. Private plans, status snapshots, mockups,
    screenshots, and roadmap notes should stay ignored.

## Investigation First

When fixing a bug or changing behavior:

1. Reproduce or identify the observed failure.
2. Trace the source of truth: domain event, stored record, projection, selector,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RojhatToptamus/task-monki](https://github.com/RojhatToptamus/task-monki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
