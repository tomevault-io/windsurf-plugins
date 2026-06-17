---
trigger: always_on
description: This file is the entry point for coding agents (Claude Code, Codex, Cursor, Gemini CLI, Aider, …) working in this repo. It complements [README.md](README.md): the README is the human walkthrough; this file is the operational reference an agent can lift commands and rules from verbatim.
---

# AGENTS.md — STRABLE for coding agents

This file is the entry point for coding agents (Claude Code, Codex, Cursor, Gemini CLI, Aider, …) working in this repo. It complements [README.md](README.md): the README is the human walkthrough; this file is the operational reference an agent can lift commands and rules from verbatim.

> **What is STRABLE?** A benchmark of 108 real-world tables with strings, plus pipelines (modular encoder → tabular learner, and end-to-end) for evaluating them. See README.md and the paper ([arXiv:2605.12292](https://arxiv.org/abs/2605.12292)) for context.

## Typical agent-handled requests

| If the user asks… | Go to |
|---|---|
| "Set up the repo / install" | [Setup](#setup) |
| "Reproduce paper figure / table N" | [Reproducing paper artifacts](#reproducing-paper-artifacts) |
| "Run STRABLE on my own model / encoder" | [Adding a new pipeline](#adding-a-new-pipeline-component) |
| "Just compare my model on the same tables" | [Use as a dataset only](#use-strable-as-a-dataset-only) |
| "Why is my run not producing output / where do results go?" | [Output layout](#output-layout) and [Common pitfalls](#common-pitfalls) |
| "Run on a single dataset / fold to test" | [Smoke test](#smoke-test) |

---

## Guardrails — do not violate

1. **Never edit `data/data_processed/`** (or `data/data_processed_FULL/`, `data/data_processed_feature_eng/`, `data/data_processed_skewness_inverse_transformation/`). These are downloaded immutable inputs. To regenerate, re-run the download script — do not hand-patch.
2. **Never commit `results/`, `results_compiled/`, `results_pics/`, `results_tables/`, `__cache__/`, `data/llm_embeding/`, `data/data_raw/`, `data/data_processed*/`.** They are runtime outputs/inputs and gitignored. If a path appears tracked, check `.gitignore` rather than `git rm`.
3. **Never hardcode absolute paths.** All paths flow through [`configs/path_configs.py`](configs/path_configs.py), which derives from `$STRABLE_ROOT` (env var) or the repo root. To override, export `STRABLE_ROOT=/your/path` — do not edit `path_configs.py`.
4. **Don't append `_default` / `_tune` to method names.** The suffix is added automatically from `--tune_indicator`. Pass `-m num-str_tabvec_xgb -ti tune`, not `-m num-str_tabvec_xgb_tune`.
5. **Don't skip the embedding-extraction step for LLM pipelines.** `script_evaluate*.py` will fail (or silently fall back) if `data/llm_embeding/<model>/…` is missing for the requested LLM encoder. See [The two-stage rule](#the-two-stage-rule-llm-pipelines).
6. **Don't introduce a new ablation folder.** The eight scripts in `scripts/evaluate_scripts/` already cover every ablation in the paper (each has a hard-coded `ABLATION` tag that determines `results/<ABLATION>/…`).

---

## Setup

```bash
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
export STRABLE_ROOT=$(pwd)
export HF_TOKEN=hf_<user-token>           # required for HuggingFace download + gated LLMs
export HF_HOME=${HF_HOME:-~/.cache/huggingface}
python data/download_datasets.py          # downloads the 108-table default variant
```

Python 3.12 required. ContextTab needs a separate install (see README §Install). GPU (`cuda`) is recommended for any LLM encoder and for TabPFN-2.5 / TabICLv2 / TabSTAR / ContextTab; everything else runs on `cpu`.

---

## The two-stage rule (LLM pipelines)

LLM-encoder pipelines run in **two stages** that share a cache. Skip this if the user only wants Tf-Idf / TargetEncoder / Tarte or an end-to-end learner (CatBoost / ContextTab / TabSTAR / Mambular).

**Stage A — pre-compute embeddings** (writes to `data/llm_embeding/<encoder>/<dataset>/`):

```bash
python scripts/embedding_extraction_scripts/script_extract_llm_embeddings.py \
    -m llm-qwen3-8b -dn all-wide -ns 3 -fi all -dv cuda -cf False -oc False
```

For the ablation variants there are sibling scripts: `..._FULL.py` and `..._feat_eng.py` (see [Ablation matrix](#ablation-matrix)).

**Stage B — run the learner** (reads the cache, runs PCA + learner, writes `results/<ABLATION>/…`):

```bash
python scripts/evaluate_scripts/script_evaluate.py \
    --save_dir benchmark_main -m num-str_llm-qwen3-8b_tabpfn \
    -dn all-wide -ns 3 -fi all -dv cuda -cf False -oc False -ti default
```

If Stage A is missing the cache, Stage B will recompute it on-the-fly — fine for one dataset but wasteful at full scale.

---

## CLI flag grammar (`scripts/evaluate_scripts/script_evaluate*.py`)

All eight `script_evaluate*.py` share this core signature. Only `script_evaluate_impute_missing_val.py` adds one extra flag (`-mvp`).

| Flag | Long | Type | Meaning |
|---|---|---|---|
| `-dn` | `--data_name` | `str+` | Dataset name(s). Use the literal `all-wide` to expand to the full 108-table list (`data_list_wide` in [`configs/exp_configs.py`](configs/exp_configs.py)). |
| `-ns` | `--n_split` | `str+` | Outer CV folds. Paper uses `3`. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [soda-inria/strable](https://github.com/soda-inria/strable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
