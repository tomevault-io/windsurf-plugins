---
trigger: always_on
description: These instructions apply to the whole repository.
---

# Agent instructions for `hermes-local-knowledge`

These instructions apply to the whole repository.

## Before editing

1. Read `README.md`, `CONTRIBUTING.md`, and `memory.md`.
2. Check `git status --short --branch`; preserve unrelated dirty work.
3. Keep changes focused. Runtime dependencies remain Python standard library only unless a documented product need justifies otherwise.
4. Never commit generated/local state or secrets.

## Documented public boundaries

- Package version is synchronized in `plugin.yaml`, `pyproject.toml`, and `hermes_local_knowledge/__init__.py`.
- The Python plugin entry point is `local_knowledge = hermes_local_knowledge.plugin`; `plugin.register` is the registration boundary.
- Registration provides exactly five native tools (`knowledge_search`, `knowledge_get`, `knowledge_neighbors`, `knowledge_feedback`, `knowledge_usage_report`) and four hooks (`pre_llm_call`, `post_tool_call`, `on_session_end`, `on_session_finalize`).
- `indexer.__all__` is exactly: `Artifact`, `Edge`, `IndexSettings`, `build_index`, `search_index`, `get_artifact`, `get_neighbors`, `main`.
- `python -m hermes_local_knowledge.cli` is the primary standalone CLI. `python -m hermes_local_knowledge.indexer` is the preserved compatibility entry point. `hermes local-knowledge` is the smaller install/doctor surface; its worker command is host-internal.
- Preserve documented configuration aliases and defaults. Do not preserve undocumented private call shapes merely because a test once patched them.
- The three router-skill copies are an intentional packaging/install exception to DRY and must remain byte-identical with one equality assertion in `tests/test_public_contract.py`.

## Current owners

- `config.py`: configuration models and resolution.
- `implicit.py`: default-enabled search-result consumption feedback.
- `artifacts.py`: whole-artifact models, collection, privacy-safe metadata, graph edges.
- `index.py`: format-4 persistence, cross-version/SQLite build locks, rebuild classification, deterministic retrieval.
- `telemetry.py`: usage/feedback persistence and reports.
- `routing.py`: bounded explicit and implicit feedback-assisted routing.
- `evaluation.py`: read-only feedback replay and metrics.
- `service.py`: managed index and telemetry lifecycle for one resolved config.
- `okf.py`: safe OKF queue, hooks, worker, validation, fenced publication.
- `plugin.py`: Hermes tools/hooks/skill/CLI registration.
- `cli.py`: standalone and Hermes CLI adapters.
- `indexer.py`: thin compatibility facade.
- `__init__.py`: package version.

## Product, ranking, and privacy invariants

- Route to whole artifacts; do not introduce chunk RAG without an explicit design change.
- FTS is the primary broad-recall path. Deterministic identity/metadata retrieval is complementary, not a replacement.
- Operational type promotion is narrow and query-gated. Quoted searches, explicit `artifact_type` filters, skill-parent lifting, and global per-parent support-doc diversity must retain their documented behavior.
- Query-terminal artifact-type promotion uses one SQLite read snapshot and one immutable legacy baseline. Eligibility is limit-independent across the complete index, requires a complete configured entity label in the exact target's ID/title/path plus a distinct topic term, never transfers identity across support siblings, and may only apply one stable target/owner move while preserving every unrelated result's relative order.
- Parent-equivalent evaluation only relates a `skill_support_doc` to its owning skill; graph neighbors are not evaluation equivalence.
- Script search text uses routing-safe metadata, never arbitrary body literals.
- Environment names may be routing signals. Environment values, MCP credential values, raw tool arguments/output, transcripts, OCR/private document text, and secret-like schema values must not enter indexed or generated artifacts.
- Keep native model-facing tool responses concise: expose only routing decisions, actionable lookup state, feedback handles, and improvement evidence. Preserve rich index/configuration diagnostics in internal telemetry and CLI/operator surfaces rather than routine agent context.
- `$HERMES_HOME/skills/.archive` is excluded from active routing.
- Feedback/evaluation data stays local. Keep public docs/tests free of raw telemetry and private content.
- Feedback-assisted routing is current-index-root-only and bounded to the latest significant explicit query/artifact rating. Among accepted current ratings, only `useful` is positive; legacy persisted `great` rows remain positive compatibility input. A newer rejection for the route or matching current query vetoes an older overlapping positive. Promote an artifact only when the current index returns it, with at most one no-longer-than-current artifact-type retry. Explicit caller-owned indexes remain unassisted.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stepanov1975/hermes-local-knowledge](https://github.com/stepanov1975/hermes-local-knowledge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
