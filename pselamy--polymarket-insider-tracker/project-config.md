---
trigger: always_on
description: Read the [constitution](.specify/memory/constitution.md), the [slice index](specs/README.md),
---

# Repository Agent Guidance

Read the [constitution](.specify/memory/constitution.md), the [slice index](specs/README.md),
and the active slice's spec, plan, tasks, contracts, and owned
[gap-register entries](specs/audit/gap-register.md) before changing the repository.
The constitution governs; specs own what/why; plans own implementation. This file
is an entry point to those authorities, not a replacement for them.

## Scope and safe effects

- This is read-only research and monitoring, not trading execution or advice.
  Detection signals are evidence for investigation, not proof of wrongdoing.
- Preserve public APIs, CLI/configuration, schemas, thresholds, and behavior unless
  an approved specification explicitly authorizes a change and migration path.
- Supported ingestion is near-real-time polling of the documented anonymous public trades
  query with all-participant coverage by default, a 5-second cadence (about 1% of the
  published limit), a durable complete-through boundary in Redis, and a visible
  `possible-data-loss` state plus durable loss events when a page cannot be proven. The
  10-minute recovery horizon is a retention and loss-detection bound, not a completeness
  guarantee. `POLYMARKET_WS_URL` is in a deprecation window: warned, never used, never
  reinterpreted. No Polymarket credential is required or sent to the trades endpoint.
- Tests and verification must not accidentally contact market, chain, or notification
  services. Real Discord/Telegram delivery requires explicit authorization. Dry runs
  must neither deliver nor poison later real-delivery deduplication state.
  The dry-run dedup-ordering gap G-018 is closed by slice 003 (see
  `specs/audit/gap-register.md`): FR-009 dry-run safety holds because
  `AlertDispatcher.dispatch` returns before any delivery-state write, proven by the
  channel-scoped tests (`tests/alerter/test_deduplication.py`,
  `tests/integration/test_end_to_end.py`) and retained in
  `specs/003-safe-observable-operation/evidence/verification.md`. Do not treat a dry
  run alone as evidence that shared delivery state is isolated — cite the gap-register
  entry and the channel-scoped tests.
- Preserve durable research assessments. Persistence failures must be observable
  without blocking an otherwise authorized alert attempt.
- Never log secrets or credential-bearing URLs. Exercise migration downgrades only
  in disposable databases, never in the configured application database.

## Spec Kit and change ownership

Follow the applicable sequence: specify → clarify → plan → reviewer-owned domain
requirements-quality checklists → tasks → analyze → human validation → implement
→ converge. Record existing approval honestly; new scope needs its own validation.

Explicitly activate the intended feature before every per-slice command. For
slice 002, from the repository root (choose the actual slice, not this example blindly):

```bash
SPECIFY_FEATURE_DIRECTORY=specs/002-reproducible-runtime .specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
```

Do not let ignored local pointer state silently choose a slice. The built-in
`checklists/requirements.md` is author-owned; domain requirements-quality checklists
are human-reviewer-owned. Agents must not mark human criteria satisfied.

Update owned tasks and gaps with actual evidence. Append corrections and verification
receipts rather than rewriting historical results to imply earlier success. Keep
unrelated concerns in separate PRs; each requested quality gate and its underlying
remediation belong together. Preserve user changes and use one writer per worktree.

## Navigation and maintainable code

Use CodeGraph before broad source searches when available: check status, initialize
or sync as needed, explore relevant symbols/callers/tests, and inspect affected code
after changes. Keep `.codegraph/` in local Git `info/exclude`, not tracked state.
If unavailable, state that briefly and use `rg`.

Prefer cohesive small functions, explicit typed boundaries, early returns, and
named domain operations. Use tables or polymorphism when they clarify real variants;
do not trade readable branches for indirection just to lower an analyzer score.
Preserve evaluation order, transactions, retries, cancellation, and error behavior.
Start behavior changes with failing regression evidence when practical; otherwise
record why and the equivalent reproducible check in the plan.

Use established libraries before rebuilding protocols. Do not reintroduce copied
tool-specific skill trees such as `.claude/skills`. Preserve the managed
`.agents/skills/speckit-*` integration; shared project guidance belongs here and in
owned specs, not duplicated global instruction payloads.

## Test doubles and behavioral coverage

Prefer real value objects and lightweight working fakes. Reusable fakes need shared
contract tests against real implementations for exercised behavior. Keep real local
database and transport tests where they provide stronger evidence.

`unittest.mock` and generic mock frameworks are prohibited. The AST policy scans
tests and root `conftest.py`, including aliases and literal dynamic imports.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pselamy/polymarket-insider-tracker](https://github.com/pselamy/polymarket-insider-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
