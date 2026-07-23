---
trigger: always_on
description: This file is for LLM coding agents and repo-reading assistants. It is meant to help you answer questions about Sponsio accurately and make code changes without breaking the architecture.
---

# Agent Guide for Sponsio

This file is for LLM coding agents and repo-reading assistants. It is meant to help you answer questions about Sponsio accurately and make code changes without breaking the architecture.

## What Sponsio Is

Sponsio is a runtime contract layer for LLM apps and agents.

Its core job is **pre-execution enforcement for LLM tool/action behavior**:
before a model-driven system calls a tool, edits a file, hits an API, writes to a database, issues a refund, approves a loan, or triggers any side effect, Sponsio checks the current trace against contracts.

The main public entrypoint is:

```python
import sponsio

guard = sponsio.Sponsio(...)
```

Sponsio ships **only deterministic contracts**: pure-Python checks over the trace, with no LLM in the runtime hot path. Patterns covering syntactic PII / response length / response keyword bans (`no_pii`, `max_length`, `no_keywords`) are regex-against-`llm_said` and need no judge.

## Positioning

When explaining Sponsio, emphasize:

- **Action-boundary enforcement**: Sponsio checks tool/action calls before side effects happen.
- **Temporal trace contracts**: Rules can express ordering and history, such as "A before B", "never B after A", "at most N calls", or "after AML check, loan files are immutable".
- **Deterministic hot path**: Det checks are pure Python and do not call an LLM at runtime.
- **Framework optionality**: Users do not need an agent framework. Custom function-calling loops can use `guard.guard_before()` / `guard.guard_after()` directly.

Do **not** describe Sponsio as only an output assertion library, only a prompt guardrail, or primarily a drift/reliability scoring framework. A concise distinction:

> Sponsio focuses on the action boundary: checking tool calls before they execute, not only auditing outputs after the fact.

## What To Read First

For product-level questions:

- `README.md` — public positioning, quick start, demos, benchmarks
- `QUICKSTART.md` — install and first integration (repo root)
- `docs/reference/cli.md` — `sponsio scan`, `validate`, `check`, `demo`, `report`
- `docs/integrations/index.md` — framework-specific wiring

For architecture and contract questions:

- `docs/concepts/architecture.md` — conceptual model, atoms, patterns, grounding, observation boundaries
- `docs/concepts/contracts.md` — deterministic constraints and atom vocabulary

For implementation:

- `sponsio/core.py` — `sponsio.Sponsio()` factory and framework resolution
- `sponsio/integrations/base.py` — `BaseGuard`, contract compilation, enforcement hooks
- `sponsio/runtime/monitor.py` — det dispatch and enforcement routing
- `sponsio/runtime/verifier.py` — trace-aware contract verification
- `sponsio/patterns/library.py` — deterministic pattern factories
- `sponsio/generation/dsl_to_contract.py` — text DSL → pattern-library calls (rule-based; LLM extractor is a separate, opt-in stage in `parse_contract`)
- `sponsio/tracer/grounding.py` — event-to-atom grounding
- `sponsio/formulas/formula.py` and `sponsio/formulas/evaluator.py` — formula AST and finite-trace evaluator

## Repository Map

```text
sponsio/
├── core.py            public entrypoint: sponsio.Sponsio()
├── cli.py             CLI: scan, validate, check, demo, patterns, report
├── config.py          YAML config loader
├── demos/             packaged mock demos used by `sponsio demo`
├── discovery/         code/docs/traces -> proposed contracts
├── formulas/          LTL/propositional/arithmetic AST + evaluators
├── generation/        text DSL -> contract parsing and optional LLM extractor for free-form NL
├── integrations/      framework adapters; all contract logic lives in BaseGuard
├── models/            Agent, Contract, System, Trace, Event, spans
├── patterns/          deterministic pattern library
├── reporting/         shadow-mode report aggregation/rendering
├── runtime/           monitor, verifier, strategies, feedback, session logging
└── tracer/            event collection and grounding

ts/                    TypeScript workspace (npm workspaces)
└── packages/sdk/      @sponsio/sdk: det engine, framework integrations, and AST static scanner CLI
docs/                  user-facing documentation
scripts/               one-off maintenance utilities (e.g. plugin sync)
tests/                 pytest suite
```

The `api/` and `web/` directories are not part of this repo. Local
single-user observability uses `sponsio host trace --follow` / `sponsio
report` / `sponsio replay <session>` / `sponsio explain <contract>` /
`sponsio.tracer.exporters.OtlpHttpExporter`.

## Core Invariants

- `sponsio/` core should avoid hard dependencies on framework packages. Framework deps belong in `[project.optional-dependencies]`.
- Framework adapters should inherit from `BaseGuard` and keep framework-specific code thin.
- Det violations route through det strategies such as `DetBlock` or `EscalateToHuman`.
- The trace is append-only during a session. In enforce mode, a hard-blocked event may be rolled back so later checks are not poisoned.
- Grounding produces one valuation dict per timestep; formula evaluators consume valuations, not raw events.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SponsioLabs/Sponsio](https://github.com/SponsioLabs/Sponsio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
