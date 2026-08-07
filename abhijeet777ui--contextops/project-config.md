---
trigger: always_on
description: > Instructions for autonomous coding agents contributing to ContextOps.
---

# AGENTS.md

> Instructions for autonomous coding agents contributing to ContextOps.

ContextOps is a deterministic structural analysis tool for LLM context. Every contribution must preserve that identity.

This document describes the project's architectural constraints, engineering philosophy, and contribution expectations for both human contributors and autonomous coding agents.

---

## Mission

ContextOps provides **deterministic static analysis** for LLM context **before inference**.

Its purpose is to measure the structural health of context — not the quality of prompts, reasoning, retrieval, or model outputs.

Every change should strengthen this mission.

---

## Core Design Principles

### 1. Determinism Is Non-Negotiable

Given identical input, ContextOps must always produce:

- Identical score
- Identical findings
- Identical recommendations
- Identical JSON output

**Never introduce:**

- Randomness or random seeds in the analysis path
- Sampling or probabilistic scoring
- Time-dependent behavior (`datetime.now()` in scoring logic)
- Network-dependent behavior
- GPU-dependent behavior

The one intentional exception is `contextops/core/roast.py` — roasts are explicitly non-deterministic by design and are excluded from the stability contract. Do not apply determinism constraints to roast selection.

---

### 2. Model Independence

ContextOps must never require:

- LLM inference of any kind
- Embedding models (dense or sparse)
- Vector databases
- GPUs or accelerators
- External API calls in the analysis pipeline

The core analysis engine (`contextops/core/`, `contextops/analyzers/`) must remain **pure Python**.

Optional integrations may exist in `contextops/integrations/` and must not be imported by the core engine.

---

### 3. Structural Scope Only

ContextOps evaluates **only structural properties** of context:

- **Redundancy** — duplicate or near-duplicate chunks
- **Density** — format overhead, whitespace waste, entropy compression
- **Structure** — context type imbalance (system, retrieval, memory, tools)
- **Concentration** — over-reliance on a small number of sources

It intentionally does **not** evaluate:

- Semantic similarity or meaning
- Prompt quality or intent
- Hallucinations or factual correctness
- Reasoning quality
- Retrieval relevance
- Output correctness

**Do not expand the scoring engine beyond structural analysis.** If a proposed feature requires understanding what the context means, it is out of scope.

---

### 4. Scoring Stability Is a Contract

Scoring changes are **breaking changes**. Before modifying:

- Scoring formulas (`contextops/core/engine.py`)
- Penalty weights or thresholds
- Aggregation or normalization logic
- Default config values in `contextops/core/config.py`

...check `STABILITY.md` to understand version implications. Changes that alter output for identical input require at minimum a **minor version bump**; formula changes require a **major version bump**.

Run the signal contract tests (`tests/test_signal_contract.py`) and stability tests before and after any engine change.

---

## Architecture

```
Input (JSON / dict / list / str)
        │
        ▼
  Normalizer  (contextops/core/normalizer.py)
        │
        ▼
  ContextBundle  (contextops/core/models.py)
        │
        ▼
  Analyzers  (contextops/analyzers/)
   ├── tokens.py        — token counting per type
   ├── redundancy.py    — LSH + Jaccard deduplication
   ├── density.py       — format overhead / entropy compression (shadow metric)
   └── structure.py     — context type ratio analysis
        │
        ▼
  Score Engine  (contextops/core/engine.py)
   ├── _calc_redundancy_penalty()   — 0–30 pts
   ├── _calc_density_penalty()      — 0–30 pts
   ├── _calc_structure_penalty()    — 0–20 pts
   └── _calc_concentration_penalty()— 0–20 pts
        │
        ▼
  Recommendations  (engine._generate_recommendations())
        │
        ▼
  AnalysisResult  (contextops/core/models.py)
        │
        ├── CLI Renderer  (contextops/cli/renderer.py)
        └── JSON Output   (AnalysisResult.to_dict())
```

**Keep analyzers independent.** An analyzer must read only its designated inputs (see the signal contract in `STABILITY.md`). Cross-axis reading is prohibited:

- `redundancy.py` reads only the `ContextBundle`
- `density.py` reads only raw text / structural properties
- `structure.py` reads only token ratios by type
- `concentration_penalty` in the engine reads only source metadata

---

## File Map

| Path | Purpose |
|------|---------|
| `contextops/core/engine.py` | Central orchestrator and scoring |
| `contextops/core/models.py` | All data models (`ContextBundle`, `AnalysisResult`, etc.) |
| `contextops/core/normalizer.py` | Input parsing into `ContextBundle` |
| `contextops/core/config.py` | Config schema and archetype presets |
| `contextops/core/roast.py` | Non-deterministic roast engine (excluded from stability) |
| `contextops/core/telemetry.py` | Local-only event logging |
| `contextops/analyzers/redundancy.py` | LSH-based redundancy detection |
| `contextops/analyzers/density.py` | Structural density signal |
| `contextops/analyzers/structure.py` | Context type ratio analysis |
| `contextops/analyzers/tokens.py` | Token counting via tiktoken |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Abhijeet777ui/contextops](https://github.com/Abhijeet777ui/contextops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
