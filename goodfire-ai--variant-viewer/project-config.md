---
trigger: always_on
description: Evo Variant Effect Explorer. Svelte 5 + DuckDB web app for visualizing Evo2 ClinVar probe predictions.
---

# variant-viewer (EVEE)

Evo Variant Effect Explorer. Svelte 5 + DuckDB web app for visualizing Evo2 ClinVar probe predictions.

## Three Components (strict boundaries, test-enforced)

```
┌──────────────────────────────────────────────────────────────────┐
│ COMPONENT 1: PIPELINE (merge.py)                                 │
│ Inputs:  ARTIFACTS/{version}/, data/variants.parquet,            │
│          data/heads.feather                                      │
│ Output:  builds/{version}/scores.parquet + heads.feather +       │
│          config.json                                             │
│ Rule:    ONLY place that filters columns (quality + parents).     │
│          NO display logic, NO viewer concepts.                   │
│          VUS scores are OPTIONAL (warn, don't crash)             │
├──────────────────────────────────────────────────────────────────┤
│ COMPONENT 2: BUILD (transform.py, build.py)                      │
│ Inputs:  builds/{version}/scores.parquet + heads.feather +       │
│          config.json                                             │
│ Output:  variants.duckdb, statistics.json, heads.json            │
│ Rule:    Adds display strings + stats. NEVER drops columns.      │
│          Asserts width >= input width.                            │
├──────────────────────────────────────────────────────────────────┤
│ COMPONENT 3: SERVE (serve.py) + FRONTEND                         │
│ Inputs:  DuckDB + heads_config from global_config                │
│ Output:  flat JSON API + UI                                      │
│ Rule:    Dumb pipe. No data transformation. No data/ reads.      │
└──────────────────────────────────────────────────────────────────┘
```

**Boundary enforcement**: `tests/test_architecture.py` enforces:
- Import boundaries (merge doesn't import transform, etc.)
- Source-of-truth rules (transform/build never drop columns, no name-pattern derivation)
- No format workarounds (no isinstance checks, no legacy fallbacks)
- merge.py is the ONLY file that filters columns

## Key files

```
cli.py              5 commands: train, extract, build, serve, ls
merge.py            Pipeline → builds/ boundary (Component 1)
transform.py        Config-driven display + stats (Component 2, zero internal imports)
build.py            Parquet → DuckDB + neighbors + UMAP (Component 2)
serve.py            DuckDB → flat JSON API (Component 3, dumb pipe)
prompts.py          Claude interpretation: build_prompt(flat, heads_config)

paths.py            Path constants — ALL paths must come from here or CLI flags

pipeline/sequence/  Sequence probe: train, extract, eval, finalize
pipeline/token/     Token probe: train, extract (with inline eval), finalize, build_labels
frontend/           Svelte 5 + Vite + ECharts (Component 3)
```

**Deleted modules** (do not recreate):
- `variant_types.py` — was an unnecessary intermediate type. `build_prompt` takes flat dict directly.
- `display.py` — display names live in `data/heads.feather` column `display`.
- `pipeline/token/positions.py` — position selection is now inside `extract.py` (`select_positions`).
- `pipeline/token/eval.py` — eval is now inline in `extract.py`, per shard. No separate eval step.

## Column naming (the contract)

- `ref_{head}` / `var_{head}` — disruption (token probe, ref vs var allele at argmax position)
- `maxpos_{head}` — genomic distance (bp) where each head's max |delta| occurs
- `eff_{head}` — effect (sequence probe, variant-level predictors)
- `gt_{head}` — ground truth database annotations (nullable)
- `pred_{head}` — categorical predictions (aa_swap, consequence)
- `pathogenicity` — overall score (derived from eff_pathogenic)

No other prefixes. `score_`, `ref_score_`, `var_score_` are **deprecated**.

## Token probe scoring contract (extract.py)

**One canonical representation: per-head probabilities.** No mixed conventions.

| Head type | Output | Columns |
|-----------|--------|---------|
| Binary (n_classes=2) | P(class=1) | 1 per head |
| Categorical (n_classes>2) | full softmax (all classes including "none") | n_classes per group |

`logits_to_probs` is the single scoring function. Binary heads (shape==2) → P(class=1). Categorical heads (shape>2) → full softmax. All class names come from `categorical_groups` in the probe config (written by train.py from heads.feather). 367 probability columns total (262 binary + 89 annotated members + 16 "none" classes).

**Loss function** (`multihead_loss_v3`):
- Equal weight per head — no `log(2)/log(K)` scaling. CE gradient norms are naturally balanced (~1.4× across class counts).
- Groups by `(kind, n_classes)` for batching, but weighted by `len(members)` per group so each HEAD gets equal gradient.
- Hierarchical children processed individually, masked to parent-positive positions.

**Position selection** (`select_positions`):
- K=6: site + 5 top positions by L2 importance
- **Site detection**: uses `dist_from_var == 0`, NOT hardcoded tensor indices. All dist=0 positions excluded from topk.
- **Per-head argmax**: `aggregate_max` returns ref/var/dist at each head's max |delta| position (`maxpos_` columns). Captures sharp single-head disruptions that L2 misses.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/goodfire-ai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
