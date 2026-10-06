---
trigger: always_on
description: validates on the other three cell lines and is tested in three settings: `unseen_ctx` (held-out
---

# PIE

PIE predicts how a perturbation changes gene expression in a cell context. For each (context,
perturbation) pair and every measured gene it outputs the probability of differential expression
(`p_de`), the log2 fold change (`lfc_pred`) and the expression-level shift (`delta_p_pred`), from
knowledge-source embeddings of the perturbation, context and genes plus pooled training evidence.

## Repository map

- `src/pie/configs/`: Hydra configs shipped in the package, one per command (`process.yaml`, `prep.yaml`, `sources.yaml`, `train.yaml`,
  `eval.yaml`, `infer.yaml`), overlays in `process/dataset/`, `prep/dataset/`, `prep/label_format/`
  and `experiment/` (the two canonical experiments).
- `src/pie/cli.py`: the `pie <command>` entry point (`process`, `prep`, `sources`, `train`, `eval`,
  `infer`); the old `pie-<command>` scripts remain as deprecated aliases.
- On HF, not in the repo: split files in `arcinstitute/PIE_splits`
  (`{"<dataset>.<context>": [perturbation, ...]}`), each dataset's context map in
  `preprocessed/contexts.yaml` (context to Cellosaurus accession, Jiang stimulations) and
  per-source perturbation aliases in `PIE_sources/<name>/aliases.yaml` (used only when a key is
  missing). `src/pie/sources/curated_aliases.yaml` is the reviewed table: `pie sources` writes
  each source's entries into its output dir, and runs saved before aliases moved read it.
- `src/pie/config.py`: strict schema of train/eval/infer (unknown keys are errors); `utils.py`:
  `common.sh` loading, env checks, config composition, logging, determinism, hashing, atomic writes.
- `src/pie/assets.py`: local/HF asset resolver, revision pinning, selective downloads,
  repository locks and completion manifests under `PIE_DATA_ROOT/hf/`.
- `src/pie/process/`: raw count h5ads to log1p expression h5ads and per-context DE parquets.
- `src/pie/prep/`: labels and expression h5ads to a preprocessed dir (`config.py` is its schema).
- `src/pie/sources/`: source format (`contract.py`), tool registry (`registry.py`), config schema
  (`config.py`), text renderers (`text/`), embedders (`embed/`) and the UniProt, STRING, DepMap and
  chemical-profile builders.
- `src/pie/data/`: preprocessed reader, splits, delta-p grid, evidence cache, samplers, datamodule.
- `src/pie/model/`: `PieModel` (Perceiver trunk, evidence module, heads) and the losses.
- `src/pie/metrics.py` (the scorer for validation and evaluation), `train.py` (Lightning module,
  trainer, checkpoints), `predict.py` (shared by `evaluate.py` and `infer.py`).
- `tests/`: mirrors `src/pie/`.

## Setup

A checkout reproduces the paper environment (`uv.lock`); users can also `pip install arc-pie`
(extras `[sources]`, `[process]`, or `[all]` for both).

```bash
uv sync --frozen                    # runtime and dev tools
uv sync --frozen --extra sources    # also the source-building tools
uv sync --frozen --extra process    # also the dataset-processing DE stage
cp common.sh.example common.sh      # then fill it in; common.sh is gitignored
```

`common.sh` sets `WANDB_ENTITY`, `WANDB_PROJECT`, `PIE_DATA_ROOT`, `PIE_RUNS_ROOT`, `PIE_CACHE_DIR`
and, optionally, `NCBI_API_KEY` and `NCBI_EMAIL`. Values are literal (no `$VAR` or `~` expansion, no
inline comments). Every command loads the first of `$PIE_ENV_FILE`, `./common.sh` and
`~/.config/pie/common.sh`; variables already set in the environment win,
and a CLI stops at once when a variable it needs is missing. The text-embedding steps also need
`OPENAI_API_KEY`: export it in your own environment, never in `common.sh`.

## CLI tools

Every command takes Hydra-style `key=value` overrides on its config in `src/pie/configs/`
(`pie process` → `process.yaml`, `pie prep` → `prep.yaml`, `pie sources` → `sources.yaml`,
`pie train` → `train.yaml`, `pie eval` → `eval.yaml`, `pie infer` → `infer.yaml`); `pie` lists the
commands and `pie <command> --help` prints the usage. Overlays are
picked with `dataset=<name>`, `label_format=<name>` or `experiment=<name>`, unknown keys are errors,
relative paths are relative to the current directory, and an existing output is an error unless you pass
`overwrite=true`.

Preprocessed dataset inputs (`data.preprocessed_dirs` in train; `preprocessed_dirs` in
eval/infer/sources) and model source inputs (`data.source_dirs`, `data.gene_text_dir`) accept
local paths or `hf://datasets/<owner>/<repo>[@revision]/<directory>` references. Bare repo IDs
are local paths; use the explicit URI for HF. The canonical experiment overlays pin full HF
commits for every dataset and source, including gene text. Dataset repos use `preprocessed/`;
`arcinstitute/PIE_sources` uses one directory per source name.

HF assets download only the selected directory to
`$PIE_DATA_ROOT/hf/datasets/<owner>/<repo>/<commit>/<directory>/`; download metadata, locks,
revision records, completion manifests and the Xet transfer cache also stay under this root.
Do not resolve HF references through `resolve_path` or turn them into `Path` before resolution:
use `resolve_asset(value, kind="preprocessed"|"source")`. Config composition and checkpoint
loading must stay free of network/download side effects. Training pins remote references before

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ArcInstitute/pie](https://github.com/ArcInstitute/pie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
