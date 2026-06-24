---
trigger: always_on
description: - `spec` is the living contract.
---

# scafld Agent Contract

## Contract

- `spec` is the living contract.
- `session` is the durable evidence ledger.
- `handoff` is transport, not source of truth.
- `review` is the adversarial completion gate.

You execute autonomously inside the contract. You do not close the task unchallenged.

## Commands

```bash
scafld init
scafld plan <task-id> --title "Title" --size small --risk low
scafld harden <task-id>
scafld harden <task-id> --mark-passed
scafld validate <task-id>
scafld approve <task-id>
scafld build <task-id>
scafld review <task-id>
scafld complete <task-id>
scafld status <task-id>
scafld list
scafld report
scafld handoff <task-id>
scafld update
```

For real review: `scafld review <task-id> --provider {codex|claude|command}`.
`--provider local` is smoke-test only and cannot satisfy `complete`.
Only an operator may use `scafld review <task-id> --human-reviewed --reason ...`.

## Source Checkout

Inside the scafld repo, use `./bin/scafld` or `go run ./cmd/scafld`. Do not use
a copied compiled binary; stale binaries can report old lifecycle state.

## Lifecycle

```text
plan -> harden -> approve -> build -> review -> complete
```

Hardening attacks the draft. Review attacks the result.
Build opens one phase at a time. After implementing the opened phase, run
`scafld build <task-id>` again to record evidence and advance.

## Do Not

- Edit outside declared scope, objectives, or invariants.
- Reconstruct lifecycle state by scraping Markdown. Use `status --json`.
- Mutate `.scafld/core/` by hand. Use `scafld update`.
- Run `--provider local` for real review.
- Cite files, commands, or review findings you have not verified.

## Prompts

`.scafld/prompts/*` overrides `.scafld/core/prompts/*` overrides built-ins.

# runx OSS Agent Guide

Canonical reference for AI coding agents working in the runx OSS workspace.
This repo uses scafld for non-trivial work, but the architecture rules are the
runx rules in `CONVENTIONS.md`, `docs/rust-kernel-architecture.md`, and
`docs/trusted-kernel-package-truth.md`.

**Key files:**

- `.scafld/config.yaml` - Validation rules, rubric weights, safety controls, profiles
- `.scafld/prompts/plan.md` - Planning mode prompt
- `.scafld/prompts/exec.md` - Execution mode prompt
- `.scafld/core/schemas/spec.json` - Spec validation schema
- `CONVENTIONS.md` - Coding standards and patterns

---

## How scafld Works

Spec-driven development: every non-trivial task becomes a machine-readable markdown specification before any code changes happen.

1. **Plan** - Analyze task, explore codebase, generate spec in `.scafld/specs/drafts/`
2. **Review** - Human reviews and approves the spec
3. **Build** - Agent executes approved spec with validation
4. **Complete** - Completed specs are marked through the scafld lifecycle

The spec is the contract. Operate autonomously within its bounds; pause for approval on deviations.

For detailed planning instructions, read `.scafld/prompts/plan.md`. For execution, read `.scafld/prompts/exec.md`.

---

## Spec Status Lifecycle

```text
draft → approved → review → completed
  ↓         ↓          ↓
(edit)   failed    cancelled
```

Valid transitions:

- `draft` → `approved` → `review` → `completed`
- active work can move to `failed` or `cancelled`
- blocked work must be recorded in the spec state and handoff

---

## Architectural Invariants

These rules must not be violated. See `config.yaml` for the canonical invariant list.

### Rust Trusted Runtime

Rust owns trusted local execution, receipt sealing, runtime policy, harness
replay, MCP, payment gates, and sandbox planning. TypeScript packages may wrap
or present those paths, but must not reintroduce local execution fallback logic.

### Pure Kernel Boundaries

Pure crates and packages stay pure. `runx-core`, `runx-contracts`,
`runx-parser`, and `runx-receipts` must not import filesystem, network,
subprocess, CLI, adapter, or runtime concerns.

### Stable Public Contracts

Public contract changes require a clean cutover through Rust-owned schemas and
fixtures. Do not add compatibility aliases, `.v2` ids, or dual-read runtime
shims for governed wire shapes.

### Generic Stateful Effects

Official skills that drive stateful hosted apps emit generic effect transition
packets. Put product identity in `effect_family` and the runner/action in
`operation`; do not add product-specific `AuthorityResourceFamily` variants or
`runx.<product>.*` packet namespaces. Stateful app memory belongs in the hosted
stateful-effect substrate and its declared reducers/views, not in OSS core
enums or bespoke runtime branches.

### No Legacy Fallbacks

No dual-reads, dual-writes, or runtime fallbacks. When changing schemas or identifiers, adopt the new scheme immediately. Use one-off migration scripts, not runtime code.

### Loop Orchestration

Long-running agent workflows are loops over governed turns, not resident kernel
loops. The loop host lives in an app, hosted service, local script, or external
orchestrator. It owns scheduling, durable loop state, wakeups, projections, and
stop policy. A runx turn is one skill or graph run with explicit inputs,
authority, `allowed_tools`, optional `context_skills`, bounded model/tool
rounds, approval gates, and one sealed receipt.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [runxhq/runx](https://github.com/runxhq/runx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
