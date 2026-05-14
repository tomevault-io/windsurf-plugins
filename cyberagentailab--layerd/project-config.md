---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LayerD is a layer decomposition method that extracts editable layers from raster graphic design images. The system uses a two-stage iterative approach:

1. **Top-layer matting**: Extracts the alpha matte of the topmost layer using BiRefNet
2. **Inpainting**: Fills in the removed content using LaMa to reconstruct the background

The main `LayerD` class orchestrates this pipeline iteratively to decompose an image into multiple layers (background + foreground layers).

## Documentation Structure

All detailed documentation is now organized in the `docs/` directory:

- **[docs/architecture.md](docs/architecture.md)** - Code architecture, design patterns, and implementation details
- **[docs/development.md](docs/development.md)** - Development setup, testing, code quality tools, and workflows
- **[docs/installation.md](docs/installation.md)** - Installation options and troubleshooting
- **[docs/pipeline.md](docs/pipeline.md)** - High-level pipeline API and orchestration
- **[docs/inference.md](docs/inference.md)** - Low-level decomposition API (CLI and Python)
- **[docs/export.md](docs/export.md)** - SVG and PSD export formats
- **[docs/training.md](docs/training.md)** - Training workflows and dataset preparation
- **[docs/evaluation.md](docs/evaluation.md)** - Evaluation metrics and usage
- **[docs/troubleshooting.md](docs/troubleshooting.md)** - Common issues and solutions

## Key Files and Components

### Pipeline Architecture

#### High-Level API (Recommended for Most Users)

- **[src/layerd/pipeline.py](src/layerd/pipeline.py)** - `LayerDPipeline` orchestration (decompose → organize → classify → export)
- **[docs/pipeline.md](docs/pipeline.md)** - Pipeline usage guide

#### Low-Level Decomposition API

- **[src/layerd/models/layerd.py](src/layerd/models/layerd.py)** - `LayerD` class with `decompose()` method
- **[src/layerd/models/helpers.py](src/layerd/models/helpers.py)** - Refinement utilities (unblend, mask operations)

### Model Implementations

- **[src/layerd/models/matting/](src/layerd/models/matting/)** - Matting model implementations
  - `base.py` - BaseMatting abstract class
  - `birefnet_matting.py` - BiRefNet implementation
  - `__init__.py` - Registry with `build_matting()` factory
- **[src/layerd/models/inpaint/](src/layerd/models/inpaint/)** - Inpainting model implementations
  - `base.py` - BaseInpaint abstract class
  - `lama_inpaint.py` - LaMa implementation
  - `__init__.py` - Registry with `build_inpaint()` factory

### Training

- **[src/layerd/matting/birefnet/](src/layerd/matting/birefnet/)** - BiRefNet training code
- **[src/layerd/configs/train.yaml](src/layerd/configs/train.yaml)** - Training configuration (Hydra)

### Evaluation

- **[src/layerd/evaluation/](src/layerd/evaluation/)** - Evaluation metrics
  - `edit_distance.py` - LayersEditDist metric
  - `dtw.py` - Dynamic Time Warping
  - `metrics.py` - Per-layer metrics (RGBL1, AlphaIoU)

### Element Organization & Classification

- **[src/layerd/postprocess/organizer.py](src/layerd/postprocess/organizer.py)** - `LayerOrganizer` for element extraction
- **[src/layerd/classification/](src/layerd/classification/)** - Element type labeling
  - `base.py` - `ElementLabeler` abstract class
  - `entropy.py` - `EntropyLabeler` implementation
  - `gradient.py` - `GradientAwareLabeler` implementation

### Export Functionality

- **[src/layerd/export/](src/layerd/export/)** - SVG and PSD export implementations
  - `base.py` - `BaseExporter` abstract class
  - `svg.py` - `SVGBuilder` implementation
  - `psd.py` - `PSDBuilder` implementation
- **[docs/export.md](docs/export.md)** - Export format guide

### OCR Support (Optional)

- **[src/layerd/ocr/](src/layerd/ocr/)** - Text detection and recognition
  - `base.py` - `BaseOCR` abstract class
  - `east_backend.py` - EAST detector (lightweight, CPU-compatible)
  - `transformers_backend.py` - GOT-OCR2 (full OCR with recognition)
  - `README.md` - OCR backend documentation

### Common Types

- **[src/layerd/types.py](src/layerd/types.py)** - Shared type definitions (`BoundingBox`, `Element`)

### Utilities

- **[tools/infer.py](tools/infer.py)** - Batch inference script
- **[tools/train.py](tools/train.py)** - Training script
- **[tools/evaluate.py](tools/evaluate.py)** - Evaluation script
- **[tools/generate_crello_matting.py](tools/generate_crello_matting.py)** - Dataset generation

## Quick Command Reference

See [docs/development.md](docs/development.md) for complete documentation.

### Testing

```bash
uv run pytest  # Run all tests

# Type checking
uv run mypy src/ tests/ \
  vendor/simple-lama-inpainting/simple_lama_inpainting/ \
  vendor/cr-renderer/src/cr_renderer/

uv run ruff check src/ tests/ vendor/  # Linting
uv run ruff format src/ tests/ vendor/ # Format code
```

**Claude Code users**: Use the custom skills for convenience:

```bash
/check         # Run all quality checks
/check mypy    # Just type checking
/fix           # Auto-fix linting and format code
```

See [.claude/skills/README.md](.claude/skills/README.md) for details.

### Inference

```bash
# High-level pipeline (recommended)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CyberAgentAILab/LayerD](https://github.com/CyberAgentAILab/LayerD) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
