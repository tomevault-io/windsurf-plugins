---
trigger: always_on
description: Python package for dimensionality reduction of protein language model (pLM) embeddings, with annotation retrieval and data export for interactive visualization at [protspace.app](https://protspace.app).
---

# protspace — Python CLI Package

Python package for dimensionality reduction of protein language model (pLM) embeddings, with annotation retrieval and data export for interactive visualization at [protspace.app](https://protspace.app).

- **Python:** >=3.12
- **License:** MIT
- **PyPI:** `pip install protspace`
- **GitHub:** https://github.com/tsenoner/protspace

## Running Commands

**Always use `uv run` to execute Python commands in this project.** Do not use bare `python` or `python3`.

```bash
# Install with dev deps (once per clone)
uv sync --group dev

# Git hooks come from husky, installed by `pnpm install` AT THE REPO ROOT — a
# Python-only setup leaves you with no pre-commit gate. Never set core.hooksPath
# by hand; husky owns it.
pnpm install

# Run tests (skip slow) — testpaths covers both protspace + protlabel
uv run pytest -m "not slow"

# Run all tests
uv run pytest

# Lint
uv run ruff check src/ packages/ tests/

# Run CLI
uv run protspace prepare -i data/sizes/phosphatase.h5:prot_t5 -m pca2 -o output --no-scores

# Run all 6 DR methods on sample data
uv run protspace prepare -i data/sizes/phosphatase.h5:prot_t5 -m "pca2,tsne2,umap2,pacmap2,mds2,localmap2" -o output --no-scores -v

# Compare UMAP with different parameters in a single run
uv run protspace prepare -i data/sizes/phosphatase.h5:prot_t5 -m "umap2:n_neighbors=15" -m "umap2:n_neighbors=50" -m pca2 -o output --no-scores
```

## CLI Commands

Single entry point: `protspace = protspace.cli.app:app`

| Command | Purpose |
|---------|---------|
| `protspace prepare` | Full pipeline: embed → reduce → annotate → bundle |
| `protspace embed` | FASTA → HDF5 embeddings (Biocentral API or local GPU/CPU via `--backend local`). Exits non-zero on an incomplete embedding; capability-limited sequences (`--max-length`, GPU OOM) are skipped and named instead |
| `protspace project` | HDF5 → dimensionality reduction |
| `protspace annotate` | Fetch protein annotations |
| `protspace bundle` | Combine projections + annotations → .parquetbundle |
| `protspace stats` | Compute projection quality statistics (annotation-based cluster-validity + faithfulness) |
| `protspace serve` | Launch Dash web frontend |
| `protspace style` | Add annotation colors/styles |
| `protspace transfer` | Fill missing annotations from nearest reference embeddings (EAT) |

### protspace prepare Usage

```bash
protspace prepare -i <input> -m <methods> -o <output> [options]

# From HDF5: protspace prepare -i embeddings.h5 -m pca2,umap2 -o output
# From FASTA: protspace prepare -i sequences.fasta -e prot_t5 -m pca2 -o output
# Local GPU/CPU embedding (offline, needs protspace[local]): protspace prepare -i seq.fasta -e prot_t5 --backend local -m pca2 -o output
# Multi-model: protspace prepare -i seq.fasta -e prot_t5,esm2_3b -m pca2 -o output
# All 12 pLMs: protspace prepare -i seq.fasta -e prot_t5,prost_t5,esm2_8m,esm2_35m,esm2_150m,esm2_650m,esm2_3b,ankh_base,ankh_large,ankh3_large,esmc_300m,esmc_600m -m pca2 -o output
# Combine datasets (same name → union): protspace prepare -i species_a.h5:prot_t5 -i species_b.h5:prot_t5 -m umap2 -o output
# Multi-embedding (different names → intersection): protspace prepare -i esm2.h5 -i prott5.h5 -m pca2 -o output
# With similarity: protspace prepare -i emb.h5 -f seq.fasta -s -m pca2,mds2 -o output
# Name override: protspace prepare -i emb.h5:custom_name -m pca2 -o output
# Parameter sweep: protspace prepare -i emb.h5 -m "umap2:n_neighbors=15" -m "umap2:n_neighbors=50" -m pca2 -o output
# Inline params: protspace prepare -i emb.h5 -m "pca2,umap2:n_neighbors=50;min_dist=0.3" -o output
# Quality stats (opt-in): protspace prepare -i emb.h5 -m pca2,umap2 --stats -o output
# Quality stats scoped to specific annotations: protspace prepare -i emb.h5 -m pca2 --stats --stats-annotation major_group,ec_number -o output
```

### protspace stats Usage


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tsenoner/protspace](https://github.com/tsenoner/protspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
