---
trigger: always_on
description: **Hydrogel-Data-Analysis** is a Python image analysis toolkit for hydrogel particle dynamics research. It combines TIFF image loading with particle tracking (trackpy) and multiple analysis methods to characterize particle motion in hydrogel and aqueous systems.
---

# AI Coding Agent Instructions for Hydrogel-Data-Analysis

## Project Overview
**Hydrogel-Data-Analysis** is a Python image analysis toolkit for hydrogel particle dynamics research. It combines TIFF image loading with particle tracking (trackpy) and multiple analysis methods to characterize particle motion in hydrogel and aqueous systems.

### Core Domains
- **Image I/O**: Load TIFF files with OME, Leica, and PCO metadata extraction
- **Metadata Management**: Standardize instrument metadata into unified `DatasetMetadata` objects
- **Particle Tracking**: Use trackpy for spot detection and trajectory linking; also imports TrackMate XML
- **Dynamics Analysis**: Three methods — MSD (mean-squared displacement), step size distributions, and SEM particle sizing
- **UI/Visualization**: Interactive parameter tuner (matplotlib-based) and SEM particle viewer (Qt-based)

## Critical Architecture Patterns

### 1. Data Pipeline: LoadedDataset → Analysis
```
TIFF → DatasetLoader → LoadedDataset (data + DatasetMetadata)
                    ↓
              Three analysis paths:
              1. trackpy_msd.main() - Spot detection → MSD
              2. MSD_Trackpy_clean.py - TrackMate XML → MSD  
              3. Schrittweiten_methode_*.py - TrackMate XML → Step size D
```

**Key Files**:
- [data_loader.py](hydro_analysis/data_loader.py): `DatasetLoader` class handles TIFF parsing
- [metadata.py](hydro_analysis/metadata.py): `DatasetMetadata` dataclass + utilities
- [trackpy_msd.py](hydro_analysis/trackpy_msd.py): Main MSD computation with spot detection
- [MSD_Trackpy_clean.py](hydro_analysis/MSD_Trackpy_clean.py): Cleaner XML→MSD pipeline
- [Particle_Parameter_Tuner.py](hydro_analysis/Particle_Parameter_Tuner.py): Interactive GUI for tuning detection params

### 2. Metadata Extraction Strategy
The loader applies **cascading metadata sources**:
1. Basic TIFF tags (Artist, Software, DateTime)
2. OME XML (pixel size, timestamps, channel names) — supports multiple OME versions
3. Leica-specific tags (stage position, Z-step)
4. PCO image metadata
5. Sidecar XML files from Data directory (pytrackmate outputs)

Each source fills missing fields; later sources don't overwrite. Example: `_populate_metadata_from_ome()` → `_populate_metadata_from_leica()` → `_populate_metadata_from_standard_tags()` → `_populate_metadata_from_sidecars()`.

**Convention**: Always preserve raw XML as `DatasetMetadata.raw_metadata` for debugging.

### 3. Dataset Kind Classification
`DatasetMetadata.infer_kind()` heuristically labels data as "FRAP", "SPT", or "FULL" based on:
- Keywords in notes + software field (case-insensitive)
- Axis counts: FRAP/bleach → "FRAP"; sparse C + T>100 → "SPT"; Z>1 + T>1 → "FULL"

This hints at analysis strategy without requiring explicit user input.

### 4. Trackpy Integration & MSD Computation
**Parameters** (from `trackpy_msd.main()`):
- `diameter`, `distance`, `minmass`: Spot detection (trackpy.locate)
- `mpp` (µm/px), `fps`: Calibration (from metadata)
- `smooth`, `radius`: Optional background subtraction (Gaussian or rolling-ball)

**Output**:
- Trajectories as pandas DataFrame with columns: `particle`, `frame`, `x`, `y`
- MSD computed via trackpy's built-in; power-law fitting via `fit_powerlaw_with_errors()`

**Files**:
- [MSD_Trackpy_clean.py](hydro_analysis/MSD_Trackpy_clean.py): XML parsing with `load_tracks_xml()`; outputs trackpy-compatible DataFrame
- [pytrackmate_MSD_XML.py](hydro_analysis/pytrackmate_MSD_XML.py): Batch analysis over multiple TrackMate XMLs; includes particle size detection
- [trackpy_msd.py](hydro_analysis/trackpy_msd.py): End-to-end: TIFF → spot detection → linking → MSD
- [MSD_Trackpy.py](hydro_analysis/MSD_Trackpy.py): Older version; legacy

### 5. Interactive Parameter Tuning
[Particle_Parameter_Tuner.py](hydro_analysis/Particle_Parameter_Tuner.py) provides a matplotlib-based GUI for optimizing trackpy parameters:
- **Real-time preview**: Shows detected spots overlaid on selected frame
- **Live linking**: Displays particle trajectories as they're linked
- **Adjustable params**: Sliders for diameter, minmass, search_range, memory, rolling-ball/Gaussian filters
- **Frame navigation**: Step through video to verify detection quality
- **Export tracks**: Save linked DataFrame to pickle when satisfied

**Usage Pattern**: Run standalone on a TIFF to find optimal parameters before batch processing.

### 6. SEM Particle Analysis
Two complementary tools for analyzing scanning electron microscopy images of hydrogel particles:
- [sem_particle_analysis.py](hydro_analysis/sem_particle_analysis.py): Core analysis logic (watershed segmentation, diameter measurement)
- [sem_particle_viewer.py](hydro_analysis/sem_particle_viewer.py): Qt GUI with histogram display, overlay visualization, and parameter adjustment

## Developer Workflows

### Running Scripts Directly
Most scripts are standalone modules with `main()` functions:
```bash
# MSD analysis from TIFF
python -m hydro_analysis.trackpy_msd

# Interactive parameter tuning
python hydro_analysis/Particle_Parameter_Tuner.py


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Qindral) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
