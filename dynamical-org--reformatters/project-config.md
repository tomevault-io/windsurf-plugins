---
trigger: always_on
description: This project contains code to reformat weather data into the Zarr v3 / Icechunk file format.
---

This project contains code to reformat weather data into the Zarr v3 / Icechunk file format.

## Approach overview

Datasets are created in 3 phases:
1. A template of the dataset, in the form of zarr metadata that is checked into the repo, is created with `uv run main <dataset_id> update-template`. This template (not in-code config) is loaded by steps 2 and 3 and drives processing and output in those steps. This approach of checking in the metadata allow us to review diffs if the structure or metadata of the dataset changes.
2. A zarr backfill is run. The backfill uses kubernetes indexed jobs to run work in parallel. When the user runs a `uv run main <dataset-id> backfill-kubernetes ...` command the metadata for the zarr is first written by the local process to the final zarr store, then a kubernetes index job is kicked off with each job index responsible for writing a portion of the zarr chunk data into the zarr archive.
3. Operational updates to the zarr are run using a kubernetes cronjob and validated by another kubernetes cronjob which runs after the update is expected to succeed. Updates use the same parallel worker model as backfills. To ensure the archive is valid to readers throughout the update, zarr v3 metadata is written only after all workers finish, and icechunk stores use a temporary branch that is atomically merged to main.

## Repository structure

```
src/reformatters/
├── __main__.py              # CLI entrypoint, dataset registry
├── common/                  # Shared utilities and base classes
│   ├── dynamical_dataset.py # DynamicalDataset base class
│   ├── template_config.py   # TemplateConfig base class
│   ├── region_job.py        # RegionJob base class
│   ├── config_models.py     # DataVar, Coordinate, etc.
│   ├── iterating.py         # Parallelization helpers (get_worker_jobs)
│   ├── kubernetes.py        # Job/CronJob definitions
│   ├── storage.py           # Storage config and store factories
│   ├── validation.py        # Dataset validators run operationally
│   └── ...                  # Other utilities
├── <provider>/              # e.g., noaa/, ecmwf/, dwd/
│   └── <model>/             # e.g., gfs/, hrrr/, ifs_ens/
│       └── <variant>/       # e.g., forecast/, analysis/
│           ├── __init__.py
│           ├── dynamical_dataset.py
│           ├── template_config.py
│           ├── region_job.py
│           └── templates/
│               └── latest.zarr/  # Checked-in zarr metadata
├── contrib/                 # Community-contributed datasets
└── example/                 # Template for new integrations

tests/                       # Mirrors src/ structure
docs/
├── dataset_integration_guide.md      # Step-by-step new dataset integration walkthrough
├── parallel_processing.md            # How parallel writes coordinate across workers
├── add_new_variable.md               # Add new variable to an existing dataset
├── validation.md                     # Run + read validation plots; data quality checklist
├── chunk_shard_layout_tool.md        # Zarr V3 chunk/shard layout optimizer
├── source_data_exploration_guide.md  # Explore/document source data structure before integration
├── ops_card.md                       # Operations: monitoring, troubleshooting, manual updates
└── staging.md                        # Run concurrent dataset versions for testing
deploy/                      # Docker and kubernetes configs
├── Dockerfile               # Container image for reformatter jobs
└── aws/                     # nodepool.yaml, create_new_aws_open_data_bucket.sh
```

- **Shared provider utilities** Check `src/reformatters/<provider>/` for shared modules (e.g., `ecmwf/ecmwf_grib_index.py`, `noaa/noaa_utils.py`).
- **Common utilities** Look for relevant utilities in `src/reformatters/common/` before implementing equivalent logic. An incomplete list: `download.py` (`http_download_to_disk`), `iterating.py` (`group_by`, `item`, `digest`), `logging.py` (`get_logger`), `pydantic.py` (`replace`, `FrozenBaseModel`), `retry.py`, `time_utils.py` (`whole_hours`).

## Core classes

Integrating a dataset requires subclassing three base classes. For a step by step walkthrough, see [docs/dataset_integration_guide.md](docs/dataset_integration_guide.md) and for complete details of what and how subclassers should implement see the commented templates in `src/reformatters/example/{dynamical_dataset|template_config|region_job}.py`.

### TemplateConfig
Base class: `src/reformatters/common/template_config.py`, commented example subclass: `src/reformatters/example/template_config.py`.

Defines the **structure** of a dataset: dimensions, coordinates, data variables, and their attributes/encodings. Key responsibilities:
- Declare `dims`, `append_dim`, `append_dim_start`, `append_dim_frequency`
- Implement `dataset_attributes`, `coords`, `data_vars`, `dimension_coordinates()`, and optionally `derive_coordinates()`
- Generates and persists zarr metadata to `templates/latest.zarr` via `update_template()`

Always regenerate the template after any metadata changes with `uv run main <dataset-id> update-template`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dynamical-org/reformatters](https://github.com/dynamical-org/reformatters) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
