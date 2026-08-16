---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Topiary (pip package `topiary-asr`) is a Python framework for ancestral sequence
reconstruction (ASR). It wraps a pipeline of external phylogenetics tools (BLAST,
Muscle5, RAxML-NG, GeneRax, PastML, Open Tree of Life) behind a pandas-dataframe-based
API and a set of `topiary-*` CLI scripts, taking a user from a handful of seed
sequences to a full alignment, phylogenetic tree, and set of reconstructed ancestors
with an HTML report.

## Development setup

- Full install (creates external binaries, conda env, etc.) is `bash install.sh`.
  This is generally **not** needed for day-to-day development/testing — it does not
  require an NCBI API key or cluster access to work locally. It creates/updates a
  conda environment (default name `topiary`).
- For most development, the `topiary` conda environment should already be present
  at `~/miniconda3/envs/topiary/` with topiary itself installed via
  `pip install -e .`, plus `pytest`, `coverage`, `flake8`, `pytest-mock`, etc.
  Run Python/pytest/CLI commands using that environment's interpreter
  (`~/miniconda3/envs/topiary/bin/python`), not the system Python.
- `install.sh` additionally compiles custom-patched RAxML-NG and GeneRax
  (`dependencies/compile-raxml-ng.sh`, `dependencies/compile-generax.sh`) — only
  needed when those external tools themselves are being changed/rebuilt or on a
  fresh machine/cluster.

## Common commands

Run tests directly with pytest from the `topiary` conda env, e.g.:

```
pytest tests/topiary/quality
pytest tests/topiary/io/test_dataframes.py::test_specific_case
```

By default, pytest **skips** tests that hit external tools/servers. These are
opt-in via custom flags (see `tests/conftest.py`), and can be combined:

- `--run-raxml` — tests that shell out to RAxML-NG
- `--run-generax` — tests that shell out to GeneRax
- `--run-blast` — tests that run local/remote BLAST
- `--run-ncbi-server` — tests that hit the live NCBI server

`bash run_all_tests.sh` runs the full suite (flake8, completeness check, coverage
with all of the above opt-in flags enabled, badge/report generation). This is slow
and touches the network/external binaries, so it should generally **not** be run by
Claude — but it's the reference for the exact test invocation syntax (including
which `--run-*` flags to pass) if that's needed.

Tests mirror the `src/topiary` package layout under `tests/topiary/` (e.g.
`src/topiary/quality/shrink.py` ↔ `tests/topiary/quality/test_shrink.py`).
`tests/completeness_crawler.py` flags source functions that have no corresponding
test.

## Architecture

### Core data structure: the topiary dataframe

Nearly everything in topiary revolves around a single pandas dataframe format
(see `docs/source/data_structures.rst`). Required columns: `name`, `sequence`,
`species`. Topiary auto-adds `keep` (bool — sequences are never deleted, just
flagged `keep=False`), `uid` (unique 10-letter id, never edit by hand), and `ott`
(Open Tree of Life taxon id). An `alignment` column holds aligned sequences (all
same length) once alignment has occurred.

A **seed dataframe** (`name`, `species`, `aliases`, `sequence`) is the small,
human-curated starting point a user provides; `topiary.io.df_from_seed` expands it
into a full topiary dataframe via BLAST.

### API convention: functions take a df, return a modified copy

Most public topiary functions have the signature `df = topiary.something(df, ...)`,
returning a modified copy (new/changed columns, or `keep` flipped to `False`) rather
than mutating in place. This is what lets each pipeline step be run, written to CSV,
manually inspected/edited, and resumed independently. See `src/topiary/__init__.py`
for the full set of top-level re-exported functions and which submodule each comes
from (`quality`, `muscle`, `opentree`, `ncbi.blast`, `raxml`, `generax`, `io`).

### Package layout (`src/topiary/`)

- `ncbi/` — BLAST (local + NCBI remote) and NCBI Entrez access, reciprocal BLAST
  orthology calls.
- `opentree/` — Open Tree of Life species-tree lookups/resolution.
- `muscle/` — Muscle5 alignment wrapper.
- `quality/` — dataset size/redundancy reduction (`shrink_dataset`) and alignment
  polishing.
- `raxml/` — RAxML-NG wrappers: model selection, ML gene tree, ancestor inference,
  bootstraps.
- `generax/` — GeneRax wrapper for gene/species tree reconciliation.
- `pastml/` — gap-state ancestral inference (parsimony).
- `io/` — reading/writing topiary dataframes and alignments, paralog pattern
  compilation.
- `pipeline/` — the three top-level pipelines that chain the above modules together
  (see below).
- `reports/` — generates the self-contained HTML result reports (`reports/cards`
  for individual result cards, `reports/assets` for static assets bundled into
  output).
- `draw/` — tree/ancestor plotting (toytree/matplotlib based).
- `cli_scripts/` — thin argument-parsing wrappers around `pipeline/` functions;
  these are the `topiary-*` console entry points defined in `pyproject.toml`.
- `_private/` — internals not part of the public API, notably
  `_private/supervisor.py`'s `Supervisor` class, which manages the standardized

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [harmslab/topiary](https://github.com/harmslab/topiary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
