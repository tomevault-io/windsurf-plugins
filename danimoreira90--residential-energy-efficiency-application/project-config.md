---
trigger: always_on
description: **Generic agent rules for any AI coding assistant working on this repo.**
---

# Residential Energy Efficiency Application — AGENTS.md

**Generic agent rules for any AI coding assistant working on this repo.**
**Version:** 1.0 | **Date:** 2026-05-10

> This file is for agents that don't read `CLAUDE.md` (Claude Code only).
> Convention emerging across Aider, Codex, Continue, Sourcegraph Cody, Cursor.
> Keeps the project agent-portable.

---

## Identity

The Residential Energy Efficiency Application is a Brazilian-first chatbot that helps residential customers understand their `conta de luz`, identify habit-based savings, and evaluate whether installing solar panels would pay back at their location and consumption profile. v1 is Streamlit-only with Claude (Anthropic API) as the conversation engine; every quantitative claim the chatbot makes is grounded in a typed Python tool call.

Owner: Daniel Moreira. Repo: github.com/danimoreira90/Residential-Energy-Efficiency-Application.

---

## Hard Rules — non-negotiable

**HR-1 — Manual commits only.**
The owner Daniel is the only one allowed to run `git add`, `git commit`, `git push`, or any GitHub PR/merge command. Agents may show staged diffs and suggest commit messages as plain text only.

**HR-2 — v1 scope is locked.**
Build only what v1 asks for. The following are explicitly **out of scope** for v1 and must not be scaffolded, designed for, or partially implemented:

- WhatsApp delivery channel (any provider — Meta Business API, Twilio, Z-API, 360Dialog).
- Live inverter integration (Growatt cloud API, SolarEdge, Enphase, Modbus TCP, anything that talks to physical hardware).
- NILM / energy disaggregation (`nilmtk` and friends).
- Smart-home control or load shifting automation.
- Multi-tenant authentication or RBAC. v1 is single-user keyed off a Streamlit session.
- Commercial / industrial customers (Group A tariffs). v1 targets Brazilian residential Group B (B1).

If a feature request implies any of the above, an agent must stop, flag it as out-of-scope per HR-2, and wait for Daniel.

**HR-3 — Applied schema migrations are immutable.**
The DuckDB schema is versioned with timestamped SQL files in `migrations/`. Any migration with a timestamp ≤ today MUST NOT be edited. New behavior = new migration with new timestamp.

**HR-4 — Test integrity.**
- CREATE new test files: permitted (TDD red-green-refactor encouraged).
- EDIT existing test files: forbidden without explicit Daniel approval + tech-debt log entry.
- DELETE existing tests: forbidden.
- Soften assertions, add `@pytest.mark.skip`, `pytest.xfail`, or `@pytest.mark.skipif(True, ...)` to existing tests: forbidden.
- Mock the world to make a test pass: forbidden. Mock external services (Anthropic API, ANEEL, NASA POWER); never mock the function under test.

**HR-5 — LLM quantitative discipline.**
The chatbot must never invent numbers. Every quantitative claim in a chatbot response must originate from a tool call. Concretely, agents implementing chatbot features MUST:

- Register each capability as a typed tool via `energia.chat.tools.registry` (see `KICKOFF.md`).
- Place all numeric computation inside the tool function — never in the system prompt or in free-text post-processing.
- Include the rule "Você nunca inventa números — toda afirmação quantitativa vem de uma chamada de ferramenta" in the system prompt and never weaken it.
- Log every tool call and its result, so we can audit the chain `user message → tool call → result → narration`.

**HR-6 — LGPD discipline on bill data.**
Brazilian energy bills contain personal data (CPF/CNPJ, full address, installation number). Agents implementing bill ingestion MUST:

- Never log bill PDFs, photos, or extracted CPF/CNPJ to stdout, files, or external services.
- Never embed user identifiers in Anthropic API request metadata.
- Store bill data only in the local DuckDB at `data/energia.duckdb` (gitignored).
- When in doubt about whether a piece of data is PII, treat it as PII.

---

## Stack (current as of 2026-05-10)

```
Runtime         python 3.11+
Package mgr     uv (uv.lock committed)
Layout          single-package src/energia/
Frontend        streamlit (st.chat_message based)
LLM             anthropic python SDK + claude-sonnet-4-6
Validation      pydantic 2 + pydantic-settings
Solar PV        pvlib + nasa power weather data
Data            pandas + duckdb (local file)
Tariff source   aneel dados abertos rest api (cached)
Tests           pytest + pytest-mock + responses (http mocking)
Lint/format     ruff (line length 100, target py311)
Type-check      pyright in strict mode (or mypy)
CI              github actions (when configured)
```

External Python deps require a one-line ADR entry in `docs/adr/` before adding.

---

## Disciplines

This project applies:

**TDD** — Test-Driven Development. Failing test before production code. Red → Green → Refactor. Standard across all non-LLM code.

**EDD** — Eval-Driven Development. For every chatbot capability (each registered tool), a capability eval suite runs before merge. Capability pass@3 ≥ 0.90 to ship; regression pass^3 = 1.00 to ship. Evals live under `evals/` as JSONL fixtures + a runner.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danimoreira90/Residential-Energy-Efficiency-Application](https://github.com/danimoreira90/Residential-Energy-Efficiency-Application) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
