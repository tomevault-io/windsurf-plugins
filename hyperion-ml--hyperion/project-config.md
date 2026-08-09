---
trigger: always_on
description: This directory contains dataset preparation classes. Their job is to convert dataset-specific metadata and media layouts into the Hyperion metadata format used by `HyperDataset`, `InfoTable` subclasses, CLIs, recipes, and training/evaluation code.
---

# AGENTS.md

This directory contains dataset preparation classes. Their job is to convert dataset-specific metadata and media layouts into the Hyperion metadata format used by `HyperDataset`, `InfoTable` subclasses, CLIs, recipes, and training/evaluation code.

Use this file together with the repository-level `AGENTS.md`. The most relevant docs are:

- `docs/hyper_dataset.rst`
- `docs/info_tables.rst`
- `docs/data_prep.rst`

## Purpose

Each data prep class should:

1. Parse one external speech dataset layout.
2. Build Hyperion metadata tables such as `RecordingSet`, `SegmentSet`, `ClassInfo`, `EnrollmentMap`, `TrialKey`, or `TrialNdx`.
3. Save a `HyperDataset` bundle under `output_dir`.
4. Register itself as a `hyperion-prepare-data` subcommand through `DataPrep.registry`.

The typical final call is:

```python
dataset = HyperDataset(
    segments=segments,
    recordings=recordings,
    classes=classes,
    enrollments=enrollments,
    trials=trials,
    sparse_trials=False,
)
dataset.save(self.output_dir)
```

## Hyperion Metadata Directory Format

A prepared dataset directory is a saved `HyperDataset` bundle. It normally contains a `dataset.yaml` file plus one file per manifest table.

Typical layout:

```text
data/my_dataset/
  dataset.yaml
  segments.csv
  recordings.csv
  speaker.csv
  gender.csv
  language.csv
  enrollment.csv
  trials.csv
```

Typical `dataset.yaml`:

```yaml
segments: segments.csv
recordings: recordings.csv
classes:
  speaker: speaker.csv
  gender: gender.csv
  language: language.csv
enrollments:
  enrollment: enrollment.csv
trials:
  trials: trials.csv
```

Only `segments` is conceptually required by `HyperDataset`. In data prep classes for speech corpora, `recordings` is almost always produced too. `classes`, `enrollments`, and `trials` are optional and task-dependent.

## Core Table Semantics

All metadata tables are `InfoTable` subclasses backed by pandas `DataFrame`s. Every table has an `id` column and uses `id` as the index.

### `SegmentSet`

`SegmentSet` is the anchor table. Every row is one utterance, clip, or speech segment.

Required column:

- `id`: unique segment id.

Common columns:

- `recording`: id of the source recording when a segment is a slice of a longer recording.
- `start`: segment start time in seconds within `recording`.
- `duration`: segment duration in seconds.
- `speaker`: speaker class id.
- `gender`: gender class id.
- `language`: language class id, preferably ISO 639-3 alpha-3 such as `eng`.
- `transcript`: original transcript text when available.
- `corpusid`: broad corpus/source identifier.
- `dataset`: dataset prep identifier, usually `self.dataset_name()`.
- `source_type`: modality/source code such as `cts`, `afv`, or `intv`.
- `original_bandwidth`: original audio bandwidth when known.

Two valid patterns are common:

- Standalone clip: `segments.id` also identifies the audio file; no `recording` or `start` is needed.
- Slice of a longer recording: `segments.recording` points to `recordings.id`, and `start`/`duration` identify the time span.

### `RecordingSet`

`RecordingSet` describes physical audio storage.

Required columns:

- `id`: unique recording id.
- `storage_path`: file path or pipe command accepted by Hyperion audio readers.

Common columns:

- `duration`: recording duration in seconds.
- `sample_freq`: detected sampling frequency.
- `target_sample_freq`: optional desired resampling target.

If each segment is stored as one audio file, `recordings.id` usually matches `segments.id`. If segments are slices, `segments.recording` points to `recordings.id`.

Use `RecordingSet(...).sort()` and compute durations with either `recs.get_durations(self.num_threads)` or `self.get_recording_duration(recs)`, depending on the local pattern.

### `ClassInfo`

`ClassInfo` stores class vocabularies. A `ClassInfo` table row is a class label, not a segment.

The class table name should match the segment column it describes:

- `segments["speaker"]` -> `classes["speaker"]` saved as `speaker.csv`.
- `segments["gender"]` -> `classes["gender"]` saved as `gender.csv`.
- `segments["language"]` -> `classes["language"]` saved as `language.csv`.

Required column:

- `id`: class id used in the corresponding `SegmentSet` column.

Common columns:

- `class_idx`: contiguous integer index when needed by training code.
- `weights`: class weights.
- dataset-specific metadata such as speaker gender, nationality, book title, or accent.

Create `ClassInfo` only for labels that are useful downstream. Do not create class tables for every incidental provenance column.

### `EnrollmentMap`

`EnrollmentMap` maps speaker verification model ids to enrollment segment ids.

On disk, the common NIST-compatible column names are:

- `modelid`
- `segmentid`

In memory, `EnrollmentMap` normalizes `modelid` to its `id` column.

Use `EnrollmentMap(df)` when constructing enrollment metadata in memory. Some older prep scripts save enrollment CSV paths directly and pass them to `HyperDataset`; both patterns exist.

### `TrialKey` and `TrialNdx`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hyperion-ml/hyperion](https://github.com/hyperion-ml/hyperion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
