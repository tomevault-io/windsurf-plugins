---
trigger: always_on
description: ComfyUI-FlowMatching-Upscaler is a ComfyUI custom node package providing:
---

# CLAUDE.md - AI Assistant Guide

## Project Overview

ComfyUI-FlowMatching-Upscaler is a ComfyUI custom node package providing:
1. **Flow Matching Progressive Upscaler** - Progressive latent upscaling for flow-matching models (like Qwen Image)
2. **DyPE for Qwen Image** - Dynamic Position Extrapolation to enable high-resolution generation beyond training resolution

The nodes are registered under **Flow Matching** and **DyPE** categories in ComfyUI.

## Repository Structure

```
ComfyUI-FlowMatching-Upscaler/
├── __init__.py              # ComfyUI node registration, exports NODE_CLASS_MAPPINGS
├── src/
│   ├── __init__.py          # Package namespace
│   ├── flow_matching_upscaler.py  # Main upscaler nodes (FlowMatchingProgressiveUpscaler, FlowMatchingStage, LatentChannelStatsPreview)
│   ├── dype_qwen_image.py   # DyPE node wrapper (DyPEQwenImage)
│   ├── qwen_spatial.py      # DyPE spatial embedding implementation (QwenSpatialPosEmbed)
│   └── rope.py              # Rotary Position Embedding utilities
├── tests/
│   ├── conftest.py          # Pytest fixtures and ComfyUI stub setup
│   ├── test_flow_matching_upscaler.py  # Upscaler unit tests
│   └── test_dype_qwen_image.py         # DyPE unit tests
├── web/
│   └── preview.js           # Frontend extension for live preview support
├── docs/
│   └── LEARNINGS.md         # Development insights and decisions
├── examples/
│   └── Method-Comparison.json  # Example ComfyUI workflow
├── pyproject.toml           # Package metadata (version, dependencies)
├── requirements.txt         # Python dependencies
├── test.sh                  # Test runner script
└── .github/workflows/publish.yml  # CI for Comfy Registry publishing
```

## Key Architecture Patterns

### Node Registration
- Nodes export `NODE_CLASS_MAPPINGS` and `NODE_DISPLAY_NAME_MAPPINGS` dictionaries
- Root `__init__.py` aggregates mappings from `src/flow_matching_upscaler.py` and `src/dype_qwen_image.py`
- Supports both package import (via `__package__`) and direct execution fallback

### ComfyUI Integration
- Uses `nodes.common_ksampler` for sampling operations
- Uses `comfy.utils.common_upscale` for latent resizing
- Uses `comfy.model_management` for VRAM management
- Uses `comfy.model_patcher.ModelPatcher` for model patching

### Memory Management
- Automatic LOW_VRAM fallback on OOM errors
- Attention budget throttling via `_STREAMING_ATTENTION_BUDGET_MB`
- Manual tensor cleanup with `del` statements after use
- OOM retry logic with cache clearing (`gc.collect()`, `soft_empty_cache`)

## Development Workflow

### Running Tests

```bash
# Using the test script (sets up venv and ComfyUI stubs)
./test.sh

# Direct pytest (requires comfy stubs in path)
pytest

# Run specific test file
pytest tests/test_flow_matching_upscaler.py -v
```

### Test Architecture
- Tests use **stub modules** in `conftest.py` to mock ComfyUI dependencies
- Stubs provide: `comfy.samplers`, `comfy.utils`, `comfy.model_management`, `comfy.model_patcher`, `comfy.model_sampling`, `nodes`
- Tests mock `common_ksampler` to capture sampler calls without real model execution

### Code Style
- Type hints used throughout (Python 3.9+ style)
- Logging via `logging.getLogger(__name__)`
- DEBUG level logging for per-channel statistics diagnostics
- Dataclasses for configuration (`StageConfig`, `_GridConfig`, `_ModelGeometry`)

## Key Implementation Details

### Flow Matching Progressive Upscaler (`flow_matching_upscaler.py`)

**Core Functions:**
- `progressive_upscale_latent()` - Resize latent using ComfyUI's upscaler
- `apply_flow_renoise()` - Flow-style linear interpolation between latent and noise
- `run_sampler()` - Bridge to ComfyUI's `common_ksampler`
- `dilated_refinement()` - Low-pass refinement via downscale-sample-upscale

**Key Classes:**
- `FlowMatchingProgressiveUpscaler` - Main multi-stage upscaler node
- `FlowMatchingStage` - Single-stage node for caching benefits
- `LatentChannelStatsPreview` - Debug visualization node

**Important Constants:**
- `_SEED_STRIDE = 0x9E3779B97F4A7C15` - Golden ratio for seed perturbation
- `_STREAMING_ATTENTION_BUDGET_MB = 256.0` - Memory budget for fallback mode

### DyPE Implementation (`qwen_spatial.py`, `dype_qwen_image.py`)

**Core Components:**
- `QwenSpatialPosEmbed` - Custom positional embedder with YaRN/NTK extrapolation
- `apply_dype_to_qwen_image()` - Patches model with DyPE embedder
- `_DyPEModelSampling` - Wrapper for sigma adjustments (non-mutating)

**Methods:** `yarn`, `ntk`, `base`

**Editing Modes:** `adaptive`, `timestep_aware`, `resolution_aware`, `minimal`, `full`

### rope.py
- `get_1d_rotary_pos_embed()` - Core RoPE with YaRN/NTK/DyPE support
- `find_correction_range()` - YaRN frequency band correction
- `linear_ramp_mask()` - Smooth interpolation between frequency treatments

## Testing Conventions

### Mocking ComfyUI
```python
def fake_common_ksampler(**kwargs):
    latent_payload = kwargs["latent"]
    result = latent_payload.copy()
    result["samples"] = torch.zeros_like(latent_payload["samples"])
    return (result,)

with mock.patch.object(fm_upscaler, "common_ksampler", new=fake_common_ksampler):
    # Test code
```

### Testing OOM Fallback
- Simulate OOM by raising `RuntimeError("CUDA out of memory")`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ttulttul/ComfyUI-FlowMatching-Upscaler](https://github.com/ttulttul/ComfyUI-FlowMatching-Upscaler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
