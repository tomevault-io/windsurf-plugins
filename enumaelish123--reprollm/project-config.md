---
trigger: always_on
description: This file is read by coding agents (Codex CLI, Cursor, and others) working in this repository. It is short on purpose; the authoritative documents are linked.
---

# AGENTS.md — Working conventions for coding agents on ReproLLM

This file is read by coding agents (Codex CLI, Cursor, and others) working in this repository. It is short on purpose; the authoritative documents are linked.

## 1. What this project is

ReproLLM is a CLI-first, local-first reproducibility toolkit for LLM research experiments: a deterministic audit (`reprollm audit`), a manifest (`reprollm.yaml`), a lockfile (`reprollm.lock`), runtime capture (`reprollm run`), semantic drift detection (`reprollm diff`), export (`reprollm export`), and an opt-in LLM-assisted discovery step (`reprollm discover`).

Principle: **LLM discovers. Rules decide. Runtime verifies.**

## 2. Read before you write

**Every development session starts from the plan documents — no session invents its own scope.**

1. `docs/plan/00_architecture_and_decisions.md` — frozen decisions (`D-nn`). Never violate one; if a task seems to require it, stop and report.
2. `docs/plan/01_specification.md` — normative CLI contract, schemas, rule catalog, redaction policy, diff semantics, test requirements. Field names and behaviors come from here, not from memory.
3. [`docs/plan/README.md`](docs/plan/README.md) — the current execution-plan index. Follow its linked execution guide and the assigned sprint/session document for task IDs, acceptance criteria, and the forbidden zone (禁区). Historical milestone plans remain under `docs/plan/M<N>_*.md`. Tasks come from the assigned document, in order; acceptance criteria are turned into tests *before* implementation; the forbidden zone is binding.

If the task description and the specification disagree, the specification wins; say so in the session report or PR.

## 3. Non-negotiables

- **No LLM in the audit path.** Only `reprollm discover` may call a model endpoint, and only behind `--experimental`/config opt-in.
- **No heavy dependencies.** Never add `torch`, `transformers`, `vllm`, `huggingface_hub`, `numpy`, or anything that pulls them. Call the HF Hub HTTP API with `httpx`. Read installed versions with `importlib.metadata`, never `import` the package.
- **No network in tests.** All `httpx` calls are mocked with `respx`; an autouse fixture fails unmocked requests. `git` and `nvidia-smi` go through `reprollm.core.proc.run_cmd` so tests can stub them.
- **Rule IDs are frozen once released.** Renaming requires an `aliases` entry. Never reuse an ID for different semantics.
- **Never write user files** except the artifacts each command owns (`init` → manifest/.reprollm; `lock` → reprollm.lock; `run` → run directory; `rules accept/ignore/add/remove/restore` → project-rules.yaml; `rules remove` additionally → immutable `.reprollm/rule-archives/<archive_id>.json` recovery copies). Never modify user code, configs, or prompts.
- **Redaction is a security boundary.** Any change to `src/reprollm/core/redaction.py` must keep 100 % branch coverage and add cases to `tests/fixtures/secrets/`. A redaction bypass is a security bug, not a normal bug.
- **Persisted artifacts contain no absolute paths, hostnames, usernames, or secrets.**
- **Deterministic output.** Same inputs → byte-identical JSON (modulo timestamp fields listed in spec §22 T-02). Sort everything you emit.
- **`schema_version` on every persisted document.** Breaking a schema requires a CHANGELOG migration note and a spec update in the same PR.

## 4. Repository map

```text
src/reprollm/
  cli/           typer commands; thin, no business logic
  schemas/       pydantic v2 models (manifest, lock, run_record, profile, project_rules, config, finding, discover, state)
  core/          paths, hashing, git, envinfo, proc, redaction, yaml_io, registry, engine, context, levels, precedence, errors
  rules/         one module per category; each rule is a @register_rule class
  profiles/      built-in *.yaml, loader.py, detect.py
  integrations/  huggingface, vllm, openai_, peft, transformers_
  lock/ run/ diff/ export/ discover/ reporters/
tests/
  unit/          mirrors src layout
  fixtures/repos/      golden repositories (materialized as git repos at test time)
  fixtures/secrets/    redaction corpus
  fixtures/hf_api/     recorded Hub responses
  fixtures/nvidia_smi/ canned outputs
schemas/         exported JSON Schema (generated; CI checks freshness)
docs/            markdown docs; docs/plan/ holds the planning documents
```

## 5. Environment and commands

```bash
uv sync --dev                       # create/refresh the environment (no optional extras exist in Beta)
uv run reprollm --help
uv run pytest -q                    # full suite, no network
uv run pytest -q tests/unit/rules   # subset
uv run ruff check . && uv run ruff format --check .
uv run mypy src/
uv run reprollm schema export --out schemas/   # after changing any schema; commit the result
```

Python ≥ 3.10. Do not use features newer than 3.10 in `src/`.

## 6. How to add things

**A rule** (`src/reprollm/rules/<category>.py`):

```python
@register_rule
class ModelRevisionPinned(Rule):
    id = "model.revision_pinned"
    category = "model"
    default_severity = Severity.CRITICAL
    min_level = 2

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EnumaElish123/ReproLLM](https://github.com/EnumaElish123/ReproLLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
