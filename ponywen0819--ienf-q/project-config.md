---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

IENF-Q (Intra-Epidermal Nerve Fiber Quantification) is an automated analysis pipeline that reconstructs complete neural fiber networks from microscopic images and sparse manual annotations. It uses classical computer vision algorithms (no ML) for interpretable, reproducible results.

## Build and Run Commands

```bash
# Install dependencies (uses uv package manager)
uv sync

# Run complete pipeline (preprocessing + reconstruction)
python test_pipeline.py

# Run preprocessing only
python tools/run_preprocessing.py \
    --label data/Label/S163-2_a.tif \
    --mask data/Mask/S163-2_a.tif \
    --image data/Original/S163-2_a.tif \
    --output-dir output/preprocessing \
    --debug

# Run reconstruction only (legacy)
python test_main_entry.py

# Run with uv
uv run python test_pipeline.py

# Run tests
pytest test/
pytest test/construction/component_analyzer/  # Run specific test module
```

## Architecture

The codebase is organized into a layered architecture with clear separation of concerns:

### Project Structure

```
src/neural_reconstruction/
├── common/              # Shared data types and utilities
│   └── data_types.py   # ComponentAnalysisResult, ConnectionGraphBuilderResult
├── core/               # Core algorithms
│   ├── preprocessing/  # Image preprocessing pipeline
│   ├── construction/   # Neural network reconstruction
│   │   ├── component_analyzer/     # Skeletonization, topology, seed extraction
│   │   ├── connection_graph_builder/  # A* path finding, graph building
│   │   ├── backbone_extractor/     # MST extraction
│   │   └── main.py                 # build_neural_network() entry point
│   └── crosses_detection/  # Epidermis crossing detection
└── ui/                 # Main pipeline integration
    └── main_pipeline.py  # NeuralReconstructionPipeline class
```

### Complete Pipeline: Preprocessing + Reconstruction

**NEW: Unified Pipeline** - Use `NeuralReconstructionPipeline` in [src/neural_reconstruction/ui/main_pipeline.py](src/neural_reconstruction/ui/main_pipeline.py) for end-to-end processing from raw images to reconstructed network. This integrates preprocessing and reconstruction into a single API.

**Lower-level API** - `build_neural_network()` in [src/neural_reconstruction/core/construction/main.py](src/neural_reconstruction/core/construction/main.py) provides direct access to the reconstruction algorithm, assuming preprocessing is already done.

### Neural Network Reconstruction Process

The reconstruction process chains together four distinct phases:

**Phase 1: Connected Components Analysis**
- Uses scikit-image's `label()` and `regionprops()`
- Identifies discrete fiber segments from binary annotations
- Filters by minimum area to remove noise

**Phase 2: Component Analysis** (`component_analyzer/`)
- **Skeletonization**: Extracts centerlines using Zhang-Suen algorithm
- **Topology Construction**: Builds graph representation using `skan` library
- **Seed Extraction**: Curvature-aware placement along skeleton
  - More seeds at bends/branches to preserve fiber geometry
  - Controlled by `segment_length` parameter

**Phase 3: Connection Graph Building** (`connection_graph_builder/`)
- **Path Finding**: A* algorithm finds connections between components
- **Cost Calculation**: Multi-factor cost function:
  - `intensity_weight`: Prefers paths along bright pixels (nerve tissue)
  - `shape_weight`: Considers path smoothness and geometry
- **Graph Assembly**: Creates `NetworkX` graph with all feasible connections

**Phase 4: Backbone Extraction** (`backbone_extractor/`)
- **MST Construction**: Kruskal's algorithm for minimum spanning tree
- **Forest Output**: Produces optimal fiber network (may have multiple trees)

### Preprocessing Pipeline

Located in [src/neural_reconstruction/core/preprocessing/pipeline.py](src/neural_reconstruction/core/preprocessing/pipeline.py), the `SkinAnalysisPipeline` class handles:

1. **Green Channel Extraction**: Neural tissue has strongest signal in green channel
2. **Morphological Operations**: Opening/closing to clean binary masks
3. **Background Correction**: Rolling ball algorithm (radius configurable)
4. **Sato Vesselness Filter** (optional): Enhances tubular/fiber structures for better detection
5. **Multi-Otsu Thresholding**: More robust than single Otsu for complex intensity distributions
6. **Mask Operations**: ROI extraction using epidermis mask with configurable dilation

### Crosses Detection Module

The `crosses_detection/` module counts nerve fibers crossing the epidermis boundary:
- `segment_detector.py`: Identifies fiber segments
- `region_labeler.py`: Labels epidermis regions
- `crossing_counter.py`: Counts crossings for quantification

## Key API Entry Points

### Complete Pipeline (Recommended)

```python
from neural_reconstruction.ui.main_pipeline import NeuralReconstructionPipeline

# Using default configuration
pipeline = NeuralReconstructionPipeline()

# From file paths
result = pipeline.run_from_files(
    label_path="data/Label/S163-2_a.tif",
    mask_path="data/Mask/S163-2_a.tif",
    image_path="data/Original/S163-2_a.tif"
)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ponywen0819/ienf_q](https://github.com/Ponywen0819/ienf_q) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
