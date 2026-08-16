---
trigger: always_on
description: This file provides guidance to coding agents when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to coding agents when working with code in this repository.

## Project overview

LogLead (Log Loader, Enhancer, Anomaly Detector) is a Python library for benchmarking log anomaly
detection algorithms and log representations. It provides custom loaders for ~10 public log datasets,
~11 log representation "enhancers" (parsers, tokenizers, n-grams, embeddings), and ~11 anomaly detection
classifiers, so a given dataset/representation/classifier combination can be swapped independently. It is
also used as a backend library by the sibling projects LogDelta and VisualLogAnalyzer, so changes to
public APIs in `loglead/` can affect those consumers.

Data is represented as [Polars](https://www.pola.rs/) DataFrames throughout (not Pandas), chosen for speed.

## Environment setup

- Python 3.9–3.12 (`.python-version` pins 3.11 for local dev). Dependency/venv management is via
  [`uv`](https://docs.astral.sh/uv/); `uv run <script>` syncs the environment from `pyproject.toml`/`uv.lock`
  automatically, installing `loglead` itself editable into `.venv` — there is no separate install step, and
  no need to fiddle with `sys.path` to make `import loglead` work.
- A `.env` file is **not** required for the normal `uv` workflow (there isn't one checked in, and none is
  needed for the smoke demos or `tests/main.py`). It only matters if you're pointing scripts at your own
  full-size dataset copies on disk:
  - `LOG_DATA_PATH` is read (via `python-dotenv`) by the "bring your own full dataset" scripts —
    `demo/RawLoader_*`, `demo/parser_benchmark/*`, `demo/saner_2024_paper/*`, `demo/unsupervised_models.py`.
    The quick demos (`demo/HDFS_samples.py`, `demo/TB_samples.py`) use bundled sample parquet files instead
    and never touch it. The downloader also doesn't use it — `downloader/download_data.py` reads `root_folder` from the YAML config (`datasets.yml`/`tests/datasets.yml`) instead.
  - See `.env.sample` for the format if you do need `LOG_DATA_PATH`.
- There are two independent, **unlinked** ways to point tooling at a data directory on disk — nothing in
  the code cross-references them, so keeping them in sync (e.g. both pointing at `~/Datasets`) is on you:
  - `LOG_DATA_PATH` in `.env` — used only by the demo scripts listed above.
  - `root_folder` in `datasets.yml`/`tests/datasets.yml` — used only by `downloader/download_data.py`
    (`download_data.py:247`), optionally overridden by its `--location` CLI flag.
- `scikit-learn` needs `gcc`/`g++` to build. The `pip`-installed package does not pull in `tensorflow`, so
  `BertEmbeddings` (`loglead/parsers/bert/`) must have TF installed manually to work.

## Common commands

Run a script with `uv run path/to/script.py` (or `python path/to/script.py` from inside that script's
directory if using a plain pip install — many scripts assume they're run from their own folder and
`os.chdir` to it).

Quick smoke tests (use small parquet samples committed under `demo/samples/`, no download needed):
```
uv run demo/HDFS_samples.py
uv run demo/TB_samples.py
```

Parser benchmark demos:
```
uv run demo/parser_benchmark/ano_detection.py
uv run demo/parser_benchmark/parsing_speed.py
```

Full test suite — downloads real datasets (see Disk space below), then runs loading, enhancing, and
anomaly-detection checks end to end; takes up to ~30 minutes:
```
uv run tests/main.py
```
`tests/main.py` chains together, in order: `downloader/download_data.py --config tests/datasets.yml`
(downloads/prepares data), then `tests/loaders.py`, `tests/enhancers.py`, `tests/anomaly_detectors.py` via
`runpy`. These are plain scripts, not a pytest suite — there's no test framework, fixtures, or `-k`
filtering; run one of the four stages directly (e.g. `uv run tests/enhancers.py`) to iterate on just that
stage once its input parquet files already exist in `<root_folder>/test_data/`. Each stage prints
`MISMATCH!` warnings if a loaded dataset's row count drifts from the `expected_length` recorded in
`tests/datasets.yml`, and raises/prints on structural problems (missing mandatory columns, null or
non-UTF-8 values) rather than asserting — read the console output to see pass/fail.

Downloading datasets directly (independent of running tests):
```
uv run downloader/download_data.py                          # everything in downloader/datasets.yml
uv run downloader/download_data.py --config tests/datasets.yml  # smaller set used by the test suite
```
Edit the `datasets:` list in the relevant YAML and set `download: false` per-entry to skip datasets you
don't need. Disk space: the full set in `downloader/datasets.yml` is ~7 GB to download and ~104 GB
unzipped (Liberty/Spirit/Thunderbird dominate at 30-38 GB each) — make sure ~110 GB is free before running
the unrestricted downloader.

There is no linter, formatter, or CI workflow configured in this repo — don't invent one unless asked.

## Architecture

LogLead is a three-stage pipeline: **Loader → Enhancer → AnomalyDetector**, connected by Polars
DataFrames with a shared column-naming convention. Understanding the convention is usually more useful
than reading any single file:

- `m_*` — mandatory/raw columns produced by a Loader directly from the source log (e.g. `m_message`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EvoTestOps/LogLead](https://github.com/EvoTestOps/LogLead) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
