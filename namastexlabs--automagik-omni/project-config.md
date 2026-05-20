---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) when working with the Automagik Omni repository.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) when working with the Automagik Omni repository.

## Context & Scope

[CONTEXT]
- Root playbook for Automagik Omni. Review this before touching code, then consult topic-specific docs under `docs/` or within `genie/` wishes.
- Automagik Omni is a multi-tenant messaging hub orchestrating WhatsApp, Discord, and future channels through a FastAPI service, background services, and channel-specific handlers.
- Code spans API routes (`src/api`), channel adapters (`src/channels`), orchestration services (`src/services`), CLI entry points (`src/cli`), and SQLAlchemy models (`src/db`).

[SUCCESS CRITERIA]
✅ Behavioral learnings applied before execution; deviations escalated through `automagik-omni-self-learn`.
✅ Changes keep Omni’s multi-tenant channels functional (WhatsApp + Discord today, guardrails for future channels).
✅ Tooling, tests, and documentation reflect Automagik Omni realities—no Hive-era residues in new work.
✅ Evidence (commands, logs, screenshots) captured in wish/Forge artefacts and referenced in Death Testaments.

[NEVER DO]
❌ Assume behavior from the former Hive codebase—validate against Omni modules instead.
❌ Touch documentation or wish files unless explicitly tasked.
❌ Run tooling outside `uv` wrappers or bypass sandbox/approval requirements.
❌ Declare completion without RED→GREEN proof and recorded validation.

## Task Decomposition

```
<task_breakdown>
1. [Discovery]
   - Load the current wish/Forge task plus supporting docs listed above.
   - Inspect relevant Omni modules (`src/api`, `src/channels`, `src/services`, `src/db`, `tests`).
   - Confirm sandbox + approval requirements, active env vars, and MCP availability.

2. [Implementation]
   - Make smallest-possible changes in existing files; keep changes Omni-focused.
   - Follow TDD: tests first via `automagik-omni-tests`, implementation via `automagik-omni-coder`.
   - Propagate configuration/env/schema updates consistently (code + docs + migrations as needed).

3. [Verification]
   - Run targeted `uv run pytest ...` suites, lint/type checks, and channel API smoke tests.
   - Capture outputs, screenshots, or SQL results as evidence.
   - Summarize in the wish Death Testament before handing back to humans.
</task_breakdown>
```

## Behavioral Learnings

[CONTEXT]
- `automagik-omni-self-learn` owns violation records and overrides inconsistent instructions.
- Read latest entries before work; treat them as highest-priority guardrails.

[SUCCESS CRITERIA]
✅ Most recent entry acknowledged explicitly in planning.
✅ Violations against learnings trigger immediate self-learn escalation with evidence.
✅ Corrections validated via observable behavior (tests, logs, approvals).

[ENTRY FIELDS]
- `date` (YYYY-MM-DD) • `violation_type` • `severity`
- `trigger` • `correction` • `validation`

## Global Guardrails

### Fundamental Rules *(CRITICAL)*
- Do exactly what the wish/Forge task requests—no scope creep.
- Edit existing files when possible; create new files only with explicit approval.
- `.claude/commands/prompt.md` defines interaction style; follow it rigorously.
- Respect naming constraints from `AGENTS.md` (no “fixed”, “v2”, etc.).

### Code Quality Standards
- Favor clear, minimal solutions (KISS/YAGNI/DRY); stick to Pythonic patterns used in `src/`.
- Deliver complete implementations—no TODOs, placeholders, or half-finished code paths.
- Prefer built-in or well-known libs already referenced in `pyproject.toml`.
- Compose behavior via functions/modules; avoid unnecessary inheritance.

### File Organization Principles
- Keep modules under 350 LOC when feasible; factor helpers into `src/utils` only if reused.
- Separate API schemas (`src/api/schemas`), services (`src/services`), and models (`src/db`).
- Keep channel-specific logic inside respective handler modules.
- Maintain import hygiene; no circular dependencies or deep relative imports.

## Critical Behavioral Overrides

### Time Estimation Ban *(CRITICAL)*
- Use phase language (“Phase 1”, “Phase 2”)—never human timelines.
- Time estimates trigger `automagik-omni-self-learn` escalation.

### UV Compliance *(CRITICAL)*
- All Python invocations go through `uv`: `uv run python`, `uv run pytest`, `uv run ruff`, etc.
- Never call `python`, `pytest`, `pip`, or `coverage` directly.
- Enforce UV-first tooling across subagents; escalate violations immediately.

### `pyproject.toml` Protection *(CRITICAL)*
- Treat `pyproject.toml` as read-only; dependency changes use `uv add` commands.
- Any manual edit constitutes a critical violation.

## Workspace & Wish System

[CONTEXT]
- `/genie/` houses wishes, reports, and knowledge; it is the single source of orchestration truth.
- Wishes evolve in place; Death Testaments close the loop with evidence.

[SUCCESS CRITERIA]
✅ Active work captured in `genie/wishes/<slug>.md` with strategy, phases, agents, and evidence log.
✅ `/wish` command drives planning; no duplicate wish docs.
✅ Every wish closure references Death Testament files in `genie/reports/`.

[NEVER DO]
❌ Create `wish-v2` docs or duplicate wish folders.
❌ Start implementation without an approved orchestration strategy.
❌ Skip Death Testament when reporting success or failure.

## Strategic Orchestration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [namastexlabs/automagik-omni](https://github.com/namastexlabs/automagik-omni) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
