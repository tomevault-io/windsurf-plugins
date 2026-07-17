---
trigger: always_on
description: This repository contains `chromstream`, a Python library for parsing, processing, plotting, and persisting on-line gas chromatography data.
---

# AGENTS.md

## Purpose

This repository contains `chromstream`, a Python library for parsing, processing, plotting, and persisting on-line gas chromatography data.

Future agents should treat this as a scientific data library first, not a generic app:

- Preserve parsing correctness over API cleverness.
- Prefer small, test-backed changes.
- Keep public objects and column conventions stable unless a breaking change is intentional.

## Repository Map

- `src/chromstream/__init__.py`: top-level package exports via star imports.
- `src/chromstream/objects.py`: core dataclasses: `Chromatogram`, `ChannelChromatograms`, `Experiment`.
- `src/chromstream/data_processing.py`: baseline correction registry, integration helpers, log merging, chromatogram splitting.
- `src/chromstream/parsers/chromeleon.py`: Chromeleon `.txt` parsing and injection-time parsing.
- `src/chromstream/parsers/agilent.py`: Agilent `.ch`, `.d`, and `.dx` parsing.
- `src/chromstream/parsers/dispatch.py`: single-file parser dispatch for supported chromatogram files.
- `src/chromstream/parsers/hdf5.py`: strict parsers for ChromStream HDF5 experiment, channel, and chromatogram files.
- `src/chromstream/parsers/other_files.py`: MTO ASCII parser and several log-file parsers.
- `src/chromstream/hdf5_common.py`: shared HDF5 helpers (scalar converters, schema gate, reserved-attr sets, per-channel/per-chromatogram group read/write) reused by the writer and parser.
- `src/chromstream/writers/hdf5_writer.py`: HDF5 export for `Experiment`, `ChannelChromatograms`, and `Chromatogram`.
- `tests/`: pytest suite with representative sample files in `tests/testdata/`.
- `docs/` and `mkdocs.yml`: documentation and notebooks, built with MkDocs Material.

## Current Architecture

### Core data model

- `Chromatogram` represents one injection on one channel.
- `ChannelChromatograms` groups many `Chromatogram` objects for one detector/channel.
- `Experiment` groups channels and optional log data.

### Important invariants

- Chromatogram data is stored as a `pandas.DataFrame`.
- The first column is assumed to be retention time.
- The second column is assumed to be signal unless a method explicitly accepts `column=...`.
- Time units are taken from `metadata["time_unit"]`.
- Signal units are taken from `metadata["Signal Unit"]` first, then `metadata["signal_unit"]`.
- Integrated result tables and log tables use a `Timestamp` column.

Many methods rely on column order rather than column names. Do not reorder columns casually.

### Parsing surface

- `parse_chromatogram(path)` in `parsers/dispatch.py` currently supports:
  - Agilent `.ch`
  - Chromeleon `.txt` only when the file content matches expected chromatogram metadata
- `parse_experiment_hdf5(path)`, `parse_channel_hdf5(path)`, and `parse_chromatogram_hdf5(path)` read the experiment, channel, and chromatogram HDF5 formats respectively. Each accepts only its exact schema (`chromstream-experiment/v0.1.0`, `chromstream-channel/v0.1.0`, `chromstream-chromatogram/v0.1.0`); schema strings live as `SCHEMA_*` constants in `objects.py`.
- The HDF5 parsers verify that the file is a ChromStream HDF5 file before attempting schema-specific parsing (shared `_require_schema` gate in `hdf5_common.py`).
- `Experiment.add_chromatogram(...)` does not use the dispatch helper for all formats:
  - `.ch` goes to `parse_agilent_ch`
  - all other file paths currently go to `parse_chromatogram_txt`
- `Experiment.add_mult_chromatograms(...)` supports:
  - Agilent `.d` directories
  - Agilent `.dx` archives
  - lists of paths or `Chromatogram` objects
- `other_files.py` contains additional parsers for:
  - MTO ASCII chromatogram exports
  - several log file formats

If you add a new chromatogram format, update the parser, the dispatch layer, and the `Experiment` helpers together.

### Processing surface

- Baseline functions are registered with `@register_baseline`.
- `list_baseline_functions()` relies on registration order.
- Peak integration uses `scipy.integrate.trapezoid`.
- `split_chromatogram()` assumes the sliced chromatogram length is divisible by `n_injections`.
- `add_log_data()` merges on nearest `Timestamp` using `pandas.merge_asof`.

### Persistence

- `Experiment.to_hdf5()`, `ChannelChromatograms.to_hdf5()`, and `Chromatogram.to_hdf5()` delegate to `write_experiment_hdf5(...)`, `write_channel_hdf5(...)`, and `write_chromatogram_hdf5(...)`.
- All three types support strict HDF5 round-tripping via the matching `write_*` / `parse_*` pair. The three writers/parsers share one layout definition through the group helpers in `hdf5_common.py`, so a change to the on-disk format happens in one place.
- HDF5 layouts:
  - Experiment: file attrs for experiment metadata, then `Channels/<channel>/injections/inj-XXXX` with datasets `retention_time`, `signal`.
  - Channel: root *is* the channel group — `name` attr + `injections/inj-XXXX`.
  - Chromatogram: root *is* the chromatogram group — `retention_time`/`signal` datasets plus reserved attrs `schema`, `channel`, `injection_time`, and optional `injection_index`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MyonicS/ChromStream](https://github.com/MyonicS/ChromStream) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
