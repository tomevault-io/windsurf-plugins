---
trigger: always_on
description: This file defines how Claude Code and Codex should explore, modify, test, and verify changes in Rezi.
---

# AGENTS.md — Multi-Agent Workflow Guide for Rezi

This file defines how Claude Code and Codex should explore, modify, test, and verify changes in Rezi.
Use `CLAUDE.md` as the canonical API and design reference; this file focuses on workflow and execution discipline.

## Project Overview

Rezi is a runtime-agnostic TypeScript TUI framework in an npm-workspaces monorepo.

| Package | Path | Purpose |
|---------|------|---------|
| `@rezi-ui/core` | `packages/core/` | Runtime-agnostic framework APIs |
| `@rezi-ui/node` | `packages/node/` | Node.js terminal backend |
| `@rezi-ui/jsx` | `packages/jsx/` | JSX parity layer over `ui.*` |
| `@rezi-ui/native` | `packages/native/` | Native engine bindings |
| `@rezi-ui/testkit` | `packages/testkit/` | Testing utilities |
| `create-rezi` | `packages/create-rezi/` | Scaffolding CLI/templates |
| `@rezi-ui/bench` | `packages/bench/` | Benchmarks and profiling |

## Mandatory Code Standards

All code changes must comply with `docs/dev/code-standards.md`.
Treat it as a merge checklist for:
- TypeScript strictness
- runtime/layout/reconciliation invariants
- callback safety and async cancellation

## Mandatory Testing Policy

All behavior changes must follow `docs/dev/testing.md`.
Treat it as the canonical testing policy for:
- behavior-first test design
- contract evidence
- fidelity selection
- degraded-capability and failure-path coverage
- deciding whether an existing test should be kept, rewritten, promoted, or removed

## Exploration Protocol

### Before Writing Any Code

Run this checklist first:

1. Read this file fully.
2. Read `CLAUDE.md` sections relevant to the task.
3. Read `docs/dev/testing.md` before changing behavior or tests.
4. Read the target file and adjacent tests before changing behavior.

### Exploration Order

1. `packages/core/src/index.ts`: public export surface.
2. `packages/core/src/widgets/ui.ts`: canonical widget factory API.
3. `packages/core/src/widgets/types/`: prop contracts and callback signatures.
4. `packages/core/src/widgets/composition.ts`: composition API and hook context.
5. `packages/create-rezi/templates/`: reference implementations.
6. `packages/core/src/**/__tests__/`: expected behavior and edge cases.

### Render Pipeline

Event routing detail to preserve:

```text
key/mouse input -> router -> wheel router (nearest scroll target)
```

Key files for this path:
- `packages/core/src/runtime/router/key.ts`
- `packages/core/src/runtime/router/mouse.ts`
- `packages/core/src/runtime/router/wheel.ts`
- `packages/core/src/runtime/router/zones.ts`

## Agent Coordination Playbook

Use this when more than one agent is touching related scope.

### Task Slicing Rules

1. Assign each agent clear ownership by path/purpose.
2. Keep one agent as integration owner for final merge and test pass.
3. Split by independent concerns:
   - API/type changes
   - runtime behavior
   - JSX parity
   - docs and skills
4. Avoid overlapping edits to the same file unless intentional pair-review is required.

### Parallel Work Contract

- Each worker reports:
  - files changed
  - behavior changed
  - tests run
  - residual risks
- Integration owner resolves conflicts and runs final validation commands.
- If two workers changed the same semantic behavior differently, escalate before merge.

### Conflict Handling

When a conflict appears:

1. Preserve canonical API from `types.ts` and `ui.ts` first.
2. Reconcile docs/examples to match merged API.
3. Re-run affected unit tests immediately after resolution.
4. Run full suite before final commit.

## Risk Triage Matrix

| Change Type | Risk Level | Required Checks |
|-------------|------------|-----------------|
| Docs-only wording update | Low | lint/grep validation |
| Widget prop rename | Medium | compile + affected tests + docs parity |
| Runtime router/reconcile/layout changes | High | full test suite + integration coverage + PTY evidence |
| Drawlist protocol changes | High | codegen + protocol tests + docs sync |
| Theme/recipe behavior changes | Medium | visual snapshots + multi-theme PTY spot-check |

Escalation rules:

- High-risk changes need explicit evidence in commit or handoff notes.
- Medium-risk changes require at least one focused test pass and one integration assertion.
- Low-risk changes still require API consistency greps when docs include signatures/examples.

## Layout Engine Baseline (Current)

See `CLAUDE.md` § Layout Engine Baseline.

## Modification Protocol

### Before Changing Code

1. Read target file end-to-end.
2. Read neighboring tests and integration tests for the module.
3. Identify if the file is in a danger zone (below).
4. Confirm required sibling updates (JSX parity, docs, tests).

### Safe Modification Zones

- `packages/core/src/widgets/ui.ts`
- `packages/core/src/widgets/types/`
- `packages/core/src/widgets/protocol.ts`
- `packages/create-rezi/templates/`
- `docs/`
- test files (`**/__tests__/*.test.ts`)
- `packages/core/src/theme/`
- `packages/core/src/ui/`
- `packages/core/src/icons/`
- `examples/gallery/`

### Danger Zones (Extra Care Required)

- `packages/core/src/runtime/commit.ts`
- `packages/core/src/runtime/reconcile.ts`
- `packages/core/src/runtime/router/wheel.ts`
- `packages/core/src/app/createApp.ts`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RtlZeroMemory/Rezi](https://github.com/RtlZeroMemory/Rezi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
