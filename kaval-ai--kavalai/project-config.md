---
trigger: always_on
description: validates every boundary and records every run in a database the operator owns.
---

# AGENTS.md

Instructions for coding agents working in this repository. Humans are welcome
to read it too; it is the short form of what a new contributor needs.

Claude Code reads `CLAUDE.md`, which carries the same rules in the form that
tool expects. Keep the two in step: a convention worth adding to one belongs in
the other.

**Before making a structural change, read
[`docs/tutorials/architecture.rst`](docs/tutorials/architecture.rst).** It
states the design commitments, the invariants that follow from them, and where
each kind of change belongs. Most "where should this live?" questions are
answered there.

## What this project is

Kaval.AI is a Python library for building agentic workflows. A workflow is a
typed graph — written as YAML or built in Python — executed by an engine that
validates every boundary and records every run in a database the operator owns.

Two components:

- **`kavalai`** — the SDK and runtime. Modules live directly in the top-level
  package (`agent.py`, `db.py`, `server.py`, …).
- **`kavalai.backoffice`** + `frontend/` — a management interface that reads
  the runtime's tables.

## Layout

| Path | Contents |
|------|----------|
| `kavalai/` | Runtime: `agent.py`, `agent_service.py`, `db.py`, `server.py`, `run_context.py`, `functionkernel.py`, `schema_parser.py` |
| `kavalai/workflow/` | Engine v2: `models.py` (the graph), `engine.py`, `builder.py`, `expressions.py`, `render.py`, `tasklog/` |
| `kavalai/llm_clients/` | OpenAI, Gemini, Anthropic, Ollama and in-browser clients behind one streaming interface; `registry.py` makes that set extensible |
| `kavalai/eval/` | Evaluation against a **running** agent server: `base.py` (`AgentEvaluator`, `EvalResult`), `simple_evaluator.py` (literal matchers), `judge_evaluator.py` (a model grades a plain-language criterion), `eval_runner.py` (YAML cases + the `kavalai-eval` console script) |
| `kavalai/rag/` | `BaseRagService` (three capability tiers), `PostgresRagService` (pgvector), `SqliteRagService` (portable file index) |
| `kavalai/tools/` | Bundled tools: browser, web search, HTTP |
| `kavalai/migrations/` | Alembic sets: `agents` and `backoffice` |
| `backoffice/`, `frontend/` | Management API and Angular UI |
| `tests/` | Pytest suite for the library; mock MCP servers in `tests/helpers/`. An example's tests live beside it under `examples/`, and `testpaths` covers both |
| `docs/`, `notebooks/` | Sphinx documentation; the five tutorial notebooks are the source of truth |
| `examples/` | Runnable examples; `green_village/` (RAG chatbot, port 25000), `bakery/` (a YAML workflow with side effects, ports 25100/25101) and `business_info_agent/` (web research: search, crawl, summarise, port 25200) each ship an `eval_cases.yaml`; `support_agent/support_agent.yaml` is the branching-DAG example the docs load |

## Invariants

Violating one of these produces a defect that is hard to see in review, so
check a change against the list before proposing it.

1. **Workflow shape changes start in `kavalai/workflow/models.py`.** The
   builder, the YAML loader, the engine, the SVG renderer and the backoffice
   all derive from those models. A capability added to the engine alone is one
   that YAML cannot express and the diagram cannot draw.
2. **`run_stream()` is the only execution path.** `run()` drains it. Do not add
   a second implementation for the non-streaming case.
3. **Per-run state belongs on `RunContext`; engine-level state belongs on the
   engine.** The token accumulator is per run and must be forwarded by
   `_branch_context`. The `FunctionKernel` and its MCP sessions are opened by
   `await engine.connect()` and released by `await engine.aclose()` — never per
   run.
4. **Library code reads no environment variables.** Only entry-point `main()`
   functions do, through `kavalai/settings.py` where two of them read the
   same names; `tests/test_config_drift.py` pins the list of modules allowed
   to touch `os.environ`. Everything else takes its configuration as an
   argument — the engine's `default_llm_model` / `default_llm_parameters`,
   `set_default_normalizer()`. The backend registries are the one bounded
   exception, and arguments still win:
   `client_factory` outranks the LLM registry, `rag_services=` outranks the RAG
   one.
   **A workflow document names a registration, never a Python path or a
   connection string.** `GET /workflow` serves it and the backoffice edits it,
   so a dotted path there would be arbitrary code execution in the agent
   server. Both are rejected when the graph loads.
5. **The ORM models are the single source of truth for the schema.** Change
   `kavalai/db.py`, then autogenerate the revision; parity tests in
   `tests/test_migrate_db.py` fail if the two diverge. Bump
   `SQLITE_SCHEMA_VERSION` on any schema change, or stale browser databases
   will not be rebuilt.
6. **The base package stays Pyodide-compatible.** No greenlet, no native
   extensions beyond the prebuilt Pyodide packages. Everything else goes in the
   `common` extra.
7. **Every boundary validates, and failures are loud.** An unresolvable prompt
   reference raises; a tool result that does not match its declared model
   raises; duplicate tool or server names raise at registration.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kaval-AI/kavalai](https://github.com/Kaval-AI/kavalai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
