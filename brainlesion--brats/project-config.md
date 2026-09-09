---
trigger: always_on
description: The BraTS orchestrator provides a typed Python API for running top-performing brain tumor segmentation and synthesis algorithms from the BraTS challenge series. Algorithms run inside Docker or Singularity containers — the package standardizes input MRI data, launches the appropriate container, and collects the output NIfTI files. Users interact with task-specific classes (e.g., `AdultGliomaPreTreatmentSegmenter`, `Inpainter`).
---

# AGENTS.md

## What is this package?

The BraTS orchestrator provides a typed Python API for running top-performing brain tumor segmentation and synthesis algorithms from the BraTS challenge series. Algorithms run inside Docker or Singularity containers — the package standardizes input MRI data, launches the appropriate container, and collects the output NIfTI files. Users interact with task-specific classes (e.g., `AdultGliomaPreTreatmentSegmenter`, `Inpainter`).

## Commands

```bash
uv sync                                    # install package and development dependencies
uv sync --group docs                       # additionally install documentation dependencies
uv run pytest                              # run test suite
uv run pytest --cov=brats                  # run with coverage
uv run ruff check .                        # lint
uv run ruff format --check .               # check formatting
uv run pre-commit run --all-files          # full pre-commit checks
uv run mkdocs build --strict               # validate the documentation site
```

The package supports Python 3.9+. The optional preprocessing integration requires
Python 3.10 or newer. Tests mock container execution; running the test suite does
not require a Docker daemon or a GPU.

## Architecture

The package follows a **Template Method** pattern centered on `BraTSAlgorithm` (ABC). The inference workflow is:

```
user NIfTI files → standardize inputs → run container (Docker/Singularity) → collect output
```

### Inheritance hierarchy

```
BraTSAlgorithm (ABC)                  # defines _infer_single / _infer_batch template
├── SegmentationAlgorithm (abstract)  # implements common input standardization
│   ├── SegmentationAlgorithmWith4Modalities (concrete)
│   │   └── 7 four-modality segmenters: AdultGliomaPreTreatment, Africa, GoAT, etc.
│   └── MeningiomaRTSegmenter         # T1C-only variant, overrides _standardize_batch_inputs
├── Inpainter                         # handles t1n-voided + mask inputs
└── MissingMRI                        # handles 3-of-4 modalities, synthesizes the 4th
```

There are eight concrete segmenters in total: seven four-modality segmenters and
the T1C-only `MeningiomaRTSegmenter`.

### Backend dispatch

Backend selection uses a Strategy pattern via dictionary dispatch in `_get_backend_runner()`:

- **Docker** (`brats/core/docker.py`): Default backend. For algorithms from 2024 and earlier, it uses the MLCube `/mlcube_io0` through `/mlcube_io3` mounts and runs `infer`. For 2025 and newer algorithms, it mounts `/input` and `/output` and uses the image's default command. Images are pulled from Docker Hub when needed.
- **Singularity** (`brats/core/singularity.py`): HPC-friendly backend. Converts Docker images to a sandbox, uses `--bind` for volume mounts, `--nv` for GPU support, and a temporary `--overlay` for writable storage. It follows the same year-dependent MLCube/native container split as Docker.

Both backends expose a `run_container()` function with the same caller signature
and both honor `cuda_devices`: Docker requests the given device IDs (split on
commas), while Singularity sets `SINGULARITYENV_CUDA_VISIBLE_DEVICES` around the
container run to restrict the GPUs exposed by `--nv`. IDs refer to host GPUs.

### Algorithm configuration (data-driven registry)

Algorithm metadata lives in `.yml` files under `brats/data/meta/` (one per challenge track). At runtime, YAML is deserialized via `dacite` into the `AlgorithmList` and `AlgorithmData` dataclass hierarchy (`MetaData`, `RunArgs`, `AdditionalFilesData`). The YAML files use anchors/aliases to deduplicate shared defaults. The metadata keys must match the public algorithm enum values in `brats/constants.py`.

Model weights and other additional files are downloaded from Zenodo on first use and cached locally under `brats/data/additional_files/`.

### Key supporting modules

| Module | Role |
|--------|------|
| `brats/constants.py` | `Algorithms` enums, file path constants, output schemas |
| `brats/utils/algorithm_config.py` | YAML → `AlgorithmData` deserialization |
| `brats/utils/data_handling.py` | `InferenceSetup` context manager, `input_sanity_check` |
| `brats/utils/zenodo.py` | Zenodo download and caching of model weights |
| `brats/utils/logging.py` | Singleton console handler, `enable()`/`disable()` |
| `brats/preprocessing.py` | Optional wrappers around `brainles_preprocessing` |

Inference expects preprocessed images. `input_sanity_check` verifies image shape and
logs warnings, but it does not perform registration, skull stripping, or defacing.

## Source of truth

- Public task classes are implemented in `brats/core/` and re-exported from `brats/__init__.py`.
- Public algorithm identifiers are enum members in `brats/constants.py`.
- Runtime algorithm metadata is stored in the ten `.yml` files under `brats/data/meta/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BrainLesion/BraTS](https://github.com/BrainLesion/BraTS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
