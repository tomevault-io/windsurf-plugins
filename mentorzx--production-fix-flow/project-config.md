---
trigger: always_on
description: Version 18.0.0 • Updated 2025-12-23
---

# PFF Agent Playbook (AGENTS.md)

Version 18.0.0 • Updated 2025-12-23

This playbook is the contract for any coding agent working inside this repo.
If any instruction conflicts with user direction, the user wins.

---

## 0. Prime Directive (read this twice)

1. **Do not change the external behavior** unless the user explicitly asks.
2. **Preserve reproducibility.** Deterministic where possible; document nondeterminism where unavoidable.
3. **Prefer mechanical refactors** over “creative” rewrites.
4. **Architecture-first:** domain/application stay pure; any filesystem/db/network/concurrency/serialization lives in `src/pff/infrastructure/**` (accessed via ports), and cross-cutting helpers live in `src/pff/shared/**`.
5. **No silent “magic.”** No import-time side effects. No hidden global state.
6. **Fail loud, fail early.** Exceptions must be specific; errors must include context.
7. **Performance matters.** Avoid accidental O(N²) and giant in-memory copies. Measure when changing hotspots.
8. **Small diffs by default.** One PR = one intention (except the mechanical flag-day cutover).
9. **Write tests for correctness.** If you fix a bug, add a regression test.
10. **Document contracts.** Docstrings in English; user-facing logs/messages in Portuguese.
11. **Think critically; don't be a yes-machine.** Evaluate every user request on its merits. If an approach is inefficient, fragile, or a better alternative exists, say so with a brief rationale before proceeding. Blind agreement is a bug — reasoned pushback with ROI analysis is expected.

### Shim Exception Policy (Limited)

- **Allowed only for external library incompatibilities** where upstream APIs are removed/changed and no compatible release is available.
- Must be **documented and localized** (single module), and include a clear log entry at `warning` level in English.
- Must be **temporary**: add a TODO with removal criteria (upstream fix or dependency update).
- **Forbidden** for project-owned logic bugs or as a substitute for proper fixes.

## 0.1 Vocabulary (token-efficient)

**Architecture terms (synonyms):** Clean/Hexagonal/Ports&Adapters = domain + ports + adapters.

- `drivers/` = inbound adapters (CLI/API/consumers)
- `infrastructure/` = outbound adapters (DB/FS/HTTP/queues)
- `application/` = use cases + ports
- `domain/` = core logic (pure)

**Principles (use abbrevs in notes):** DRY, SoC, SRP, LoD, KISS, YAGNI.

**Smells (trigger words):**
Duplicated Code; Long Method; Large/God Class; Switch/if-cascade; Primitive Obsession; Shotgun Surgery; Feature Envy; Data Clumps.

**Canonical refactor moves (use these names in plans/PR notes):**
Extract Function/Class; Inline Function/Class; Move Function/Field; Hide Delegate; Remove Middle Man;
Introduce Parameter Object; Replace Conditional with Polymorphism; Decompose Conditional; Guard Clauses.

**Rule:** describe refactors as `1 smell -> 1–3 moves` (max). No essays.

---

## 1. Project overview & architecture

This repo is the production-grade PFF platform focused on:

- DSLFM-KGC training and evaluation
- Probabilistic Circuits (PC2) integration
- Stacking/ensemble orchestration + HPO (Optuna)
- Data quality validation (telecom domain)

Repository map:

- `config/` – YAML specs (tunable knobs live here; code reads configs, not the other way around).
- `data/models/` – Real KG assets (**read-only**; tests must not depend on these).
- `outputs/` – Canonical home for generated artifacts (models, metrics, plots, benches).
- `logs/` – Runtime logs (rotated/dated).
- `src/pff/drivers/` – Composition roots / entrypoints (CLI, API, Celery, HPO).
- `src/pff/application/` – Use cases + ports (orchestration; defines interfaces).
- `src/pff/domain/` – Core business/ML logic (DSLFM-KGC, PC2, anomaly scoring, rules).
- `src/pff/infrastructure/` – Adapters (DB/Postgres, filesystem, queues, external services).
- `src/pff/shared/` – Cross-cutting code used by **2+ production consumers** (strictly curated).
- `scripts/` – Operational scripts (kept thin; long-term they migrate into `src/pff/drivers/` as needed).
- `scripts/lint/` – Unified lint/guardrail pipeline (`lint_repo.py`, `log_lint.py`, `guardrail.py`). Run via `poetry run python scripts/lint/lint_repo.py --fix`.
- `tests/` – Unit/integration/e2e + golden masters + architecture tests.
- `deprecated/` – Legacy modules. Avoid for new code.

### src-layout (packaging hygiene)

- This repo uses `src/` layout (`src/pff/**`). Treat changes here as a top-level structure change: requires codemod + arch tests + editable install workflow.
- Rationale: tests/imports should exercise the installed package, not the working directory copy.

**Rule:** `src/pff/domain/**` and `src/pff/application/**` MUST NOT touch filesystem/network/DB/concurrency primitives. Side effects live in `src/pff/infrastructure/**` and are reached through ports; composition happens in `src/pff/drivers/**`.

---

## 2. Setup & canonical commands

- Install:
  - `poetry install`
- Run lint:
  - `poetry run ruff check .`
- Run formatting:
  - `poetry run ruff format .`
- Run smoke tests:
  - `poetry run pytest -q`
- Smoke suites (examples; pick the smallest relevant):
  - `poetry run pytest tests/audit/test_eval_protocol.py -q`
  - `poetry run pytest tests/data/test_kg_data_quality.py -q`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mentorzx/production-fix-flow](https://github.com/Mentorzx/production-fix-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
