---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Chokepoint is a Python framework for expressing AI agent authorization policies as
deterministic, code-defined predicates evaluated by the runtime, never by the
LLM itself. Every guarded tool call is intercepted before execution (pre-hook),
and optionally after execution (post-hook), against a set of `Policy` objects.
The core principle: an LLM never sees, interprets, or reasons its way around a
policy — policies are plain Python, evaluated entirely outside the model's
context, by a deterministic runtime chokepoint every tool call passes through.

Positioning: Chokepoint is the authorization-and-reversibility layer of an agent
harness, not a full harness. It doesn't run a plan→act→observe loop, manage
context/memory, or execute tools itself — it decides *whether* a tool call may
proceed and *what to do* if it can't (block, escalate, or auto-undo). It's
framework-agnostic middleware, meant to sit between an agent's tool-calling
loop (LangGraph, CrewAI, a hand-rolled loop, ...) and the tools themselves.

The package is published as `chokepoint` (`pip install chokepoint`).

## Commands

This project uses **uv**, not raw pip/venv.

```bash
uv sync --extra all            # install the project + dev + every optional integration
uv sync                        # no extras — mirrors the CI job that exercises the
                               # graceful-degradation paths; run this before claiming
                               # something works without the extras installed
uv run pytest -q               # run the full test suite (sync + async, via pytest-asyncio)
uv run pytest --cov=chokepoint --cov-report=term-missing   # with coverage (CI gates at 90%)
uv run pytest tests/core/test_reversible.py::test_permanent_blocks -q  # single test
uv run ruff check .            # lint (line-length=110, py311 target)
uv run ruff check --fix .      # autofix
uv run ruff format .           # format (checked in CI)
uv run mypy src/chokepoint       # strict type check
uv build                       # build sdist + wheel
uv run --group docs python scripts/build_docs.py   # assemble docs/ (markdown + API stubs)
uv run --group docs mkdocs build --strict          # docs must build clean
uv run python examples/clinical.py        # run the worked example
uv run python examples/async_tool.py      # run the async worked example
uv run python examples/real_escalation_handlers.py   # Slack/webhook/CLI escalation handlers
uv run chokepoint report --agent examples/clinical.py             # CLI: policy report
uv run chokepoint report --agent examples/clinical.py --format mermaid
uv run chokepoint lint examples/clinical.py
uv run chokepoint replay <event_id>
uv run chokepoint repl --agent examples/clinical.py
```

## Architecture

### The evaluation engine is the single chokepoint — with a sync and an async twin

`src/chokepoint/_engine.py:evaluate_call()` (and its async sibling
`evaluate_call_async()`) is the one place every guarded tool call actually
runs through. `@guard` (`core/decorator.py`), `ChokepointInterceptor.call()`,
and `ChokepointInterceptor.acall()` (`core/interceptor.py`) are thin call-sites
that build a `GuardContext`, gather the right `Policy` list, and hand off to
one of these two functions — they do not duplicate any
pre/post/escalate/undo/ledger/OTEL logic themselves. When changing how a
decision is made, recorded, or enforced, `_engine.py` is almost always the
file to edit (usually in **both** `evaluate_call`/`evaluate_call_async` — they
intentionally mirror each other line-for-line where possible), not the
call-sites.

Pipeline per call: build `GuardContext` → evaluate pre-hook rules across all
active policies → resolve the worst failure (BLOCK > ESCALATE > ALLOW
precedence, see `decisions.pick_decision`) → if ESCALATE **and** the mode is
`enforce`, call the resolved `EscalationHandler` (with `timeout_s` actually
enforced — see below) →
execute the tool (unless blocked in `enforce` mode) → evaluate post-hook rules
→ on a post-BLOCK, auto-invoke `ReversibleAction.undo` if the call wrapped one
(a failure here still records the ledger event — see below) → record one
`LedgerEvent` per hook that had a qualifying decision (always for
BLOCK/ESCALATE/log-only-ALLOW; sampled for an aggregate "everything passed"
ALLOW) → emit a `chokepoint.evaluate` OTEL span and metrics alongside it.

### Only `enforce` mode may have side effects beyond recording

`dry_run` and `observe` exist to answer "what would these policies do?" against
real traffic. That is worthless if asking the question pages a human, so
`_engine._unresolved_escalation` short-circuits every ESCALATE outside
`enforce`: the decision is still resolved to `ESCALATE`, still recorded to the
ledger, still spanned — but no `EscalationHandler` is contacted, so nothing is
posted to Slack, no approval webhook fires, and nothing blocks on `input()`.
The reason text is suffixed so the ledger distinguishes "denied" from "never
asked". Same rule for the other two side-effecting steps, which were already
gated: non-enforce modes never raise `GuardBlocked` and never auto-undo.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pedromuracchini/chokepoint](https://github.com/pedromuracchini/chokepoint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
