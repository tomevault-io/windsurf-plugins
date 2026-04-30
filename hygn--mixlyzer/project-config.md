---
trigger: always_on
description: Working notes for AI/automation agents editing the Mixlyzer codebase.
---

# AGENTS (Mixlyzer)

Working notes for AI/automation agents editing the Mixlyzer codebase.

## Run & Environment
- Python 3.12+.
- Dependency management uses `uv`.
- Install deps with `uv sync`.
- Launch the app with `uv run python -m app.main`.
- Keep root-level `config.json` available. `load_cfg()` reads and rewrites it.
- Keep root-level `ffmpeg.exe` available on Windows, or provide FFmpeg on `PATH` for `core/audio/decoder.py`.
- Do not run the app as administrator on Windows; `app/main.py` explicitly blocks it.

## Repo Map
- `app/`: Qt entrypoint and top-level window wiring.
  - `main.py`: app bootstrap, dark palette, font loading, library-version migration flow.
  - `window.py`: `AppWindow`, worker orchestration, EventBus wiring, External Sync, metronome, model updates.
- `core/`: shared runtime/state/integration layer.
  - `config.py`: dataclass config schema, defaults, `load_cfg()`.
  - `model.py`: shared `DataModel` and `GlobalParams`.
  - `event_bus.py`: Qt signal hub.
  - `analysis_worker.py`: subprocess wrapper for full-track analysis.
  - `segment_reanalysis_manager.py` / `segment_reanalysis_worker.py`: subprocess-based partial reanalysis.
  - `library_handler.py`: SQLite metadata store plus BPM/key segment tables and transition search.
  - `analysis_lib_handler.py`: NPZ feature store keyed by track UID.
  - `external_sync.py`: memory-based external deck sync / transport lockout.
  - `player.py`, `timeline.py`, `taskmanager.py`: playback, viewport timeline, worker/task UI state.
  - `audio/`: decode + low-level audio helpers.
- `analyzer_core/`: DSP and editor-side reanalysis algorithms.
  - `global_analyzer.py`: full analysis pipeline and persistence handoff.
  - `beat/`: BPM / beatgrid analysis.
  - `key/`: chroma/key analysis.
  - `self_correlation/`: JumpCUE detection.
  - `editor/`: beatgrid, key, JumpCUE reanalysis helpers used by worker subprocesses.
- `ui/`: Qt widgets/dialogs.
  - `pane.py`: `MainPane` composition.
  - `library.py`: library table, search, transition search, export/reanalyze/edit actions.
  - `beatgrid_edit_panel.py`, `edit_song.py`, `export.py`, `cfgwindow.py`, `workers.py`: editing/export/settings/task dialogs.
- `views/`: pyqtgraph view plugins registered through `views.base.REGISTRY`.
- `utils/`: labels, colors, fonts, JumpCUE extraction/render helpers, keystrip utilities, waveform helpers.
- `third_party/`: export adapters such as Rekordbox XML generation.
- `migration/`: library schema/version migrations.
- `build/`: PyInstaller spec.
- `docs/`: localized docs (`README.ko.md`, `README.ja.md`).
- `assets/`: icons, fonts, sounds.
- `library/`: user library data directory created/used at runtime. Treat as user data.
- `debug/`: generated debug artifacts; do not assume they are source assets.

## App Flow
- Entry is `app.main:main()`.
- On startup the app:
  - loads `config.json`,
  - ensures the library directory exists,
  - checks `library/VERSION`,
  - runs migration steps from `migration/` if the on-disk library version is behind `CURRENT_LIBRARY_VERSION`,
  - creates and shows `AppWindow`.
- `AppWindow` creates the shared `EventBus`, `DataModel`, `TimelineCoordinator`, `PlayerController`, `ExternalSyncController`, `SegmentReanalysisManager`, and `MainPane`.
- UI state is mostly signal-driven. Feature/property/album-art signals are notifications only; listeners read current values from the shared `DataModel`.

## Analysis Flow
- Track load requests enter through `EventBus.sig_request_load_track` and end up in `AppWindow.analyze_file()`.
- Full-track analysis runs in a spawned subprocess via `core.analysis_worker.AnalysisWorker`.
- The worker calls `analyzer_core.global_analyzer.precompute_features(...)`.
- Current pipeline in `global_analyzer.py`:
  - decode/load audio,
  - optional HPSS split,
  - low/mid/high envelope extraction plus waveform min/max buffers,
  - BPM / beatgrid analysis,
  - beatgrid offset application,
  - JumpCUE detection,
  - beat-synchronous chroma extraction,
  - key analysis,
  - GUI-buffer normalization via `core.adapters.normalize_gui_buffers`,
  - persistence to SQLite (`LibraryDB`) and NPZ (`FeatureNPZStore`).
- Cached-track loads can skip analysis and load metadata/features directly from `library.db` + NPZ.
- Reanalyze-track requests (`sig_reanalyze_requested`) force a fresh full analysis and overwrite stored results.

## Segment Reanalysis
- Partial reanalysis is orchestrated by `core.segment_reanalysis_manager.SegmentReanalysisManager`.
- Work is offloaded to a subprocess via `SegmentReanalysisWorker`.
- Supported modes currently include:
  - beat segment reanalysis,
  - static key selection reanalysis,
  - dynamic full key reanalysis,
  - JumpCUE reanalysis.
- Results are pushed back into the in-memory `DataModel` and editor widgets; they are not a separate analysis system.

## Data & Persistence
- Track metadata lives in `library/library.db` in the `tracks` table.
- BPM and key transitions are also persisted in:
  - `track_bpm_segments`
  - `track_key_segments`
- Numeric feature arrays live in `{libpath}/{uid}.npz` through `FeatureNPZStore`.
- Track identity across DB and NPZ is the `uid` field stored in SQLite.
- `DataModel` holds the live in-memory track state:
  - `features`
  - `properties`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hygn/Mixlyzer](https://github.com/hygn/Mixlyzer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
