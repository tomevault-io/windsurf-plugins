---
trigger: always_on
description: This file provides guidance to coding agents working in this repository.
---

# AGENTS.md

This file provides guidance to coding agents working in this repository.

## Commands

```bash
# Install in editable mode
pip install -e .

# Run tests
pytest tests/

# Run a single test file
pytest tests/test_class_manager.py

# Lint
ruff check src tests

# Format
black src tests

# CLI — initialize environment (downloads checkpoints)
hawkears init [--dest <path>]

# CLI — analyze recordings
hawkears analyze <input_path> -o <output_path> [options]

# Launch the desktop GUI
hawkears-gui

# Extract/compile Qt interface translations
hatch run hawkears:translations
hatch run hawkears:translations-compile
```

The local Hatch environment may fail to resolve while an unreleased BriteKit
version is being developed. In that case, use the existing environment binaries
for checks rather than changing dependency constraints merely to run tests.

## Architecture

HawkEars is a bioacoustic classifier with both a CLI and a PySide6 desktop GUI.
It sits on top of **BriteKit** (a PyTorch ML inference library) and adds
species-specific filtering, occurrence filtering, and heuristics.

### Core data structure: frame map

The central artifact throughout the pipeline is a 2D NumPy array `[num_frames, num_classes]` called the **frame map**, holding per-frame confidence scores for each species. Heuristics and filters modify it in-place; it is finally converted to labeled time intervals by BriteKit's `Predictor.get_frame_labels()`.

### Data flow

```
hawkears analyze (CLI)
  → get_config()          loads YAML defaults + device-specific overrides
  → Analyzer.__init__()
      ├─ ClassManager     parses model checkpoint class names; applies include/exclude lists
      └─ OccurrenceManager  loads eBird occurrence data for geographic/date filtering
  → Analyzer.run()
      └─ ThreadPoolExecutor (N threads, each with its own Predictor)
          ├─ Predictor.get_overlapping_scores()   → raw frame_map
          ├─ CanadaHeuristicsManager.process_recording()
          │     LowBandHeuristics → SoundAlikeHeuristics → BoostScoreHeuristics → LowerScoreHeuristics
          ├─ _update_frame_map()   zeros excluded/rare classes
          └─ return structured detections and/or write Audacity / CSV / Raven output
```

The programmatic `analyze()` API can use `return_results=True` and a
`progress_callback`. The GUI uses this path with `rtype=None`: it does not parse
CSV output. `AnalysisResult` and related records live in
`hawkears/core/analysis_result.py`. Preserve CLI defaults and behavior when
changing the API.

### Key modules

| Module | Role |
|--------|------|
| `hawkears/core/analyzer.py` | Main orchestrator; multi-threaded inference loop |
| `hawkears/core/class_manager.py` | Species metadata; include/exclude filtering; name/code lookups |
| `hawkears/core/occurrence_manager.py` | eBird geographic + date filtering |
| `hawkears/core/config.py` | Config dataclasses extending BriteKit's `BaseConfig` |
| `hawkears/core/config_loader.py` | YAML loading with device-specific overrides; global cache |
| `hawkears/heuristics/canada/main.py` | Chains the four heuristic handlers |
| `hawkears/commands/_analyze.py` | Click command + `analyze()` API entry point |
| `hawkears/commands/_init.py` | Downloads/extracts model checkpoints from GitHub releases |
| `hawkears/core/analysis_result.py` | Structured inference results and progress notifications |
| `hawkears/gui/ui/main_window.py` | Main handwritten PySide6 UI and page flow |
| `hawkears/gui/database/` | SQLite project schema, migrations, records, and repositories |
| `hawkears/gui/services/analysis_runner.py` | Background inference and result persistence |
| `hawkears/gui/services/location_catalog.py` | Hierarchical administrative-area lookup |
| `hawkears/gui/services/spectrogram.py` | Decibel-scaled review spectrogram generation |

## Desktop GUI

- A `.hawkears` project is a SQLite database. It stores project settings,
  recordings, selected species, immutable analysis-run snapshots, detections,
  revision history, and review annotations.
- Schema migrations are ordered SQL files in
  `src/hawkears/gui/database/migrations/`. Never edit an already released
  migration; add the next numbered migration.
- Inferred, imported, and manual detections share one normalized model.
  Editing species or time/frequency bounds appends a `detection_revision` so
  original inference/import values remain available for reporting.
- Use repositories instead of placing SQL in widgets. SQLite connections are
  short-lived and thread-local.
- Inference and spectrogram generation run in `QThread`s. Do not perform audio,
  model, or long database work on the GUI thread. Analysis cancellation is
  cooperative: active recordings finish, partial detections are saved, and no
  additional recordings are started.
- The Results page reads current detection revisions and review state from the
  database. The Review page writes verdicts/notes and species corrections, then
  advances through the current visible Results ordering.
- Review spectrograms use audio parameters from `yaml/default.yaml`, a 10-second
  context, `decibels=True`, and `skip_cache=True` to avoid processing an entire
  recording just to display one detection.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jhuus/HawkEars](https://github.com/jhuus/HawkEars) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
