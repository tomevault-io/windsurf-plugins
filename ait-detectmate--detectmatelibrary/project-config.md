---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DetectMateLibrary is a Python library for log processing and anomaly detection. It provides composable, stream-friendly components (parsers and detectors) that communicate via Protobuf-based schemas. The library is designed for both single-process and microservice deployments.

## Development Commands

```bash
# Install dependencies and pre-commit hooks
uv sync --dev
uv run prek install

# Run tests
uv run pytest -q
uv run pytest -s                                      # verbose with stdout
uv run pytest --cov=. --cov-report=term-missing       # with coverage
uv run pytest tests/test_foo.py                       # single test file

# Run linting/formatting (all pre-commit hooks)
uv run prek run -a

# Recompile Protobuf (only if schemas.proto is modified)
protoc --proto_path=src/detectmatelibrary/schemas/ \
  --python_out=src/detectmatelibrary/schemas/ \
  src/detectmatelibrary/schemas/schemas.proto

# Scaffold a new component workspace
mate create --type <parser|detector> --name <name> --dir <target_dir>
```

## Architecture

### Data Flow

```
Raw Logs → Parser → ParserSchema → Detector → DetectorSchema (Alerts)
```

All data flows through typed Protobuf-backed schema objects. Components are stateful and support an optional training phase before detection.

### Core Abstractions (`src/detectmatelibrary/common/`)

- **`CoreComponent`** — base class managing buffering, ID generation, and training state
  - **`CoreParser(CoreComponent)`** — parse raw logs into `ParserSchema`
  - **`CoreDetector(CoreComponent)`** — detect anomalies in `ParserSchema`, emit `DetectorSchema`
- **`CoreConfig`** / **`CoreParserConfig`** / **`CoreDetectorConfig`** — Pydantic-based configuration hierarchy (`extra="forbid"`)

#### Component Lifecycle (FitLogic)

Each `process()` call passes through a state machine controlled by config fields:

| Config field | Meaning |
|---|---|
| `data_use_configure` | How many items to run through `configure()` before training. `None` = skip. |
| `data_use_training` | How many items to run through `train()` before detection. `None` = skip. |

Phases in order: **CONFIGURE → TRAIN → DETECT** (phases are skipped when the corresponding field is `None`).

State control enums allow overriding automatic transitions: `TrainState.KEEP_TRAINING` / `STOP_TRAINING` and `EnumState.KEEP` / `STOP_CONFIGURE` on the component's `fit_logic`.

#### CoreDetectorConfig: EventsConfig

Detectors use a nested `events` structure to select which variables to track per event ID, plus `global_instances` for event-ID-independent variables (e.g., hostname, level):

```yaml
detectors:
  MyDetector:
    method_type: new_value_detector
    auto_config: false          # true = auto-discover variables from training data
    persist:                    # optional — omit to disable state saving
      path: ./state             # base path; detector name is appended automatically
      interval_seconds: 300     # save every N seconds
      events_until_save: null   # also save after N ingested events (null = disabled)
      auto_load: false          # restore saved state on construction
      storage_options: {}       # fsspec credentials (S3, Azure, GCS, etc.)
    events:
      login_failure:            # named event ID (string) or integer EventID
        instance_label:         # arbitrary instance name
          params: {}
          variables:
            - pos: pid          # named wildcard label OR integer position in ParserSchema.variables[]
              name: pid
          header_variables:
            - pos: Type         # key in ParserSchema.logFormatVariables{}
              params: {}
    global:                     # event-ID-independent instances (GLOBAL_EVENT_ID = "*")
      global_monitor:
        header_variables:
          - pos: Level
            params: {}
```

Named event IDs (strings) and named variable positions require templates loaded from a CSV with an `EventId` column. Call `TemplateMatcher.compile_detector_config(config)` to resolve names to integers at setup time.

Use `generate_detector_config()` (`src/detectmatelibrary/common/_config/_compile.py`) to build this programmatically:

```python
from detectmatelibrary.common._config import generate_detector_config

config = generate_detector_config(
    variable_selection={1: ["var_0", "var_1"]},
    detector_name="MyDetector",
    method_type="new_value_detector",
)
```

Load/save configs via `BasicConfig.from_dict(d, method_id=...)` and `.to_dict(method_id=...)` for YAML round-trip compatibility.

After training completes, detectors with `auto_config=False` automatically call `validate_config_coverage()` (`src/detectmatelibrary/common/detector.py`), which logs warnings when configured EventIDs or variable positions were never observed in training data. This catches config/data mismatches early — check logs after the training phase when adding new detector configs.

### Schema System (`src/detectmatelibrary/schemas/`)

- `BaseSchema` wraps generated Protobuf messages with dict-like access (`schema["field"]`)
- Key schemas: `LogSchema`, `ParserSchema`, `DetectorSchema`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ait-detectmate/DetectMateLibrary](https://github.com/ait-detectmate/DetectMateLibrary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
