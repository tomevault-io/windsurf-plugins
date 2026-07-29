---
trigger: always_on
description: This is the **IGN LiDAR HD Processing Library** - a Python library for processing French IGN LiDAR HD data into machine learning-ready datasets with building Level of Detail (LOD) classification support.
---

# GitHub Copilot Instructions for IGN LiDAR HD Dataset

## Project Overview

This is the **IGN LiDAR HD Processing Library** - a Python library for processing French IGN LiDAR HD data into machine learning-ready datasets with building Level of Detail (LOD) classification support.

**Version:** 3.0.0  
**Language:** Python 3.8+  
**Type:** Data Processing & ML Pipeline Library

## Core Technologies

- **LiDAR Processing:** laspy, lazrs, NumPy, SciPy
- **ML/Scientific:** scikit-learn, NumPy, PyTorch (optional)
- **GPU Acceleration:** CuPy, RAPIDS cuML, FAISS (optional)
- **Configuration:** Hydra, OmegaConf
- **Geospatial:** Shapely, GeoPandas, Rasterio, Rtree
- **Testing:** pytest

## GPU Development Environment (CRITICAL)

**ALWAYS use the `ign_gpu` conda environment for GPU feature implementation and testing.**

### GPU Testing Rule

When implementing or testing GPU features:

```bash
# ALWAYS use this command for GPU-related work
conda run -n ign_gpu python <your_script.py>

# Examples:
conda run -n ign_gpu python scripts/benchmark_gpu.py
conda run -n ign_gpu python -m pytest tests/test_gpu_*.py
conda run -n ign_gpu python -c "import cupy; print('GPU OK')"
```

### GPU Environment Details

- **Environment**: `ign_gpu` (conda)
- **GPU Libraries**: CuPy, RAPIDS cuML, RAPIDS cuSpatial, FAISS-GPU
- **Purpose**: All GPU acceleration, CUDA operations, GPU benchmarks
- **Base environment**: Should NOT be used for GPU testing (missing libraries)

### When to Switch to ign_gpu

✅ **Use ign_gpu for**:

- Implementing GPU-accelerated features
- Running GPU benchmarks
- Testing RAPIDS cuML/cuSpatial code
- Validating FAISS-GPU operations
- Any code that imports: `cupy`, `cuml`, `cuspatial`, `faiss-gpu`

❌ **Don't use ign_gpu for**:

- CPU-only code
- Pure NumPy/SciPy operations
- Standard development/testing
- Documentation generation

## Project Architecture

### Module Structure

```
ign_lidar/
├── core/              # Core processing orchestration
│   ├── processor.py         # Main LiDARProcessor class
│   ├── classification/      # Classification logic
│   ├── memory.py           # Memory management
│   ├── performance.py      # Performance monitoring
│   └── tile_stitcher.py    # Tile stitching
│
├── features/          # Feature computation
│   ├── orchestrator.py     # FeatureOrchestrator (unified API)
│   ├── feature_computer.py # Feature computation engine
│   ├── compute/            # Low-level compute functions
│   ├── strategies.py       # Strategy pattern for CPU/GPU
│   └── mode_selector.py    # Automatic mode selection
│
├── preprocessing/     # Data preprocessing
│   ├── outliers.py        # Outlier removal
│   └── augmentation.py    # RGB/NIR augmentation
│
├── io/                # Input/Output operations
│   ├── laz.py            # LAZ file handling
│   ├── metadata.py       # Metadata management
│   └── wfs_ground_truth.py # WFS ground truth fetching
│
├── config/            # Configuration management
│   ├── schema.py         # Config schema (Hydra)
│   └── defaults.py       # Default configurations
│
├── datasets/          # PyTorch datasets
│   └── multi_arch_dataset.py
│
└── cli/               # Command-line interface
    └── main.py
```

### Key Design Patterns

1. **Strategy Pattern:** CPU/GPU feature computation (`strategy_cpu.py`, `strategy_gpu.py`, `strategy_gpu_chunked.py`)
2. **Factory Pattern:** Optimization factory for adaptive processing
3. **Orchestrator Pattern:** `FeatureOrchestrator` unifies feature management
4. **Configuration Pattern:** Hydra-based hierarchical configuration

## Serena MCP Code Intelligence (CRITICAL)

**ALWAYS use Serena MCP tools for code exploration and modification.** Serena provides semantic code intelligence that understands the structure and relationships in your codebase.

### When to Use Serena MCP

**Code Exploration (REQUIRED before making changes):**

- ✅ Use `mcp_oraios_serena_get_symbols_overview` to understand file structure before reading
- ✅ Use `mcp_oraios_serena_find_symbol` to locate specific classes, functions, or methods
- ✅ Use `mcp_oraios_serena_find_referencing_symbols` to understand where code is used
- ✅ Use `mcp_oraios_serena_search_for_pattern` for flexible regex-based searches across files

**Code Modification (PREFERRED over manual edits):**

- ✅ Use `mcp_oraios_serena_replace_symbol_body` to update entire functions/classes
- ✅ Use `mcp_oraios_serena_insert_before_symbol` to add imports or new code
- ✅ Use `mcp_oraios_serena_insert_after_symbol` to append new functions/classes
- ✅ Use `mcp_oraios_serena_rename_symbol` for safe refactoring across the codebase

**Memory Management:**

- ✅ Use `mcp_oraios_serena_list_memories` to check available project knowledge
- ✅ Use `mcp_oraios_serena_read_memory` to load relevant project context
- ✅ Use `mcp_oraios_serena_write_memory` to document important patterns for future use

### Serena Workflow Example

```
# 1. Activate project
mcp_oraios_serena_activate_project("/mnt/d/Users/Simon/OneDrive/Documents/GitHub/IGN_LIDAR_HD_DATASET")

# 2. Check onboarding status
mcp_oraios_serena_check_onboarding_performed()

# 3. List available memories

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imagodata/IGN_LIDAR_HD_DATASET](https://github.com/imagodata/IGN_LIDAR_HD_DATASET) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
