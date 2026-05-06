---
trigger: always_on
description: VistaDream is a Flux-based diffusion model for image outpainting and 3D scene generation, integrated with Rerun for visualization. The project uses Pixi for package management and CUDA 12.9 for GPU acceleration.
---

# VistaDream AI Coding Instructions

## Project Overview
VistaDream is a Flux-based diffusion model for image outpainting and 3D scene generation, integrated with Rerun for visualization. The project uses Pixi for package management and CUDA 12.9 for GPU acceleration.

## Core Architecture

### Model System (`src/vistadream/flux/`)
- **Entry Point**: Multiple CLI interfaces (`cli.py`, `cli_fill.py`, `cli_control.py`, `cli_redux.py`) 
- **Model Loading**: `util.py` contains model configurations and loading functions
- **Inference Pipeline**: `sampling.py` handles diffusion sampling, `model.py` defines Flux transformer
- **Model Variants**: `flux-dev`, `flux-schnell`, `flux-dev-fill`, `flux-dev-depth`, `flux-dev-canny`
- **Memory Management**: All CLIs support `--offload` to move models between CPU/GPU for memory efficiency

### Key Pattern: Model Offloading
```python
# Standard offloading pattern used throughout
if offload:
    ae = ae.to(torch_device)  # Move to GPU for encoding
    # ... use model ...
    ae = ae.cpu()             # Move back to CPU
    torch.cuda.empty_cache()  # Clear CUDA memory
    model = model.to(torch_device)  # Move next model to GPU
```

### 3D Scene Generation (`src/vistadream/ops/gs/`)
- **Frame System**: `Frame` class handles camera parameters, RGB, depth, and inpainting masks
- **Gaussian Splatting**: `Gaussian_Scene` manages 3D Gaussians for scene representation
- **Training Loop**: `GS_Train_Tool` optimizes Gaussians using RGB/SSIM losses
- **Critical**: Always set `frame.inpaint_wo_edge` before adding to scene to avoid NoneType errors

### VistaDream Pipeline (`src/vistadream/api/vistadream_pipeline.py`)
- **Main Pipeline**: Combines outpainting → depth prediction → 3D scene generation
- **Trajectory Generation**: Uses `_generate_trajectory()` for camera movement
- **Rerun Integration**: Dynamic blueprint updates with `logged_cam_idx_list`
- **Frame Processing**: Inpaints every 5th frame, adds to Gaussian scene

### API Layer (`src/vistadream/api/`)
- **Outpainting API**: `flux_outpainting.py` - standalone outpainting with Rerun
- **VistaDream Pipeline**: `vistadream_pipeline.py` - full 3D pipeline 
- **Configuration Pattern**: Uses `@dataclass` configs with tyro for CLI generation
- **Image Processing**: Ensures dimensions are multiples of 32, handles megapixel limits

### Operations (`src/vistadream/ops/`)
- **Flux Integration**: `flux.py` wraps Flux models for inpainting
- **Trajectory Generation**: `trajs/` module for camera movement (spiral, wobble, interp)
- **Scene Utilities**: `utils.py` for point cloud operations and depth processing
- **Visual Check**: `visual_check.py` renders videos from Gaussian scenes

## Type Annotations with JAXTyping & Beartype

### Runtime Type Checking with Beartype
The project uses beartype for runtime type validation, automatically activated in `src/vistadream/__init__.py`:
```python
from beartype.claw import beartype_this_package
beartype_this_package()
```
- Full runtime type checking enabled across all modules
- No need to manually add `@beartype` decorators
- Automatic validation of function parameters and return types

### Array Type Annotations with JAXTyping
**Critical**: Every array must specify both dtype and shape using jaxtyping, and variables should be annotated at assignment time:
```python
from jaxtyping import Float, UInt8, Bool, BFloat16
import torch
import numpy as np

# Variable assignment annotations (PEP 526 style)
rgb_hw3: UInt8[np.ndarray, "h w 3"] = ...
depth: Float[np.ndarray, "h w"] = ...
mask: Bool[np.ndarray, "h w"] = ...
latents: BFloat16[torch.Tensor, "batch channels latent_height latent_width"] = ...
intrinsics: Float[np.ndarray, "3 3"] = ...
```

### Function Signatures with JAXTyping
Follow the established pattern from `ops/flux.py` and `ops/gs/basic.py`:
```python
def __call__(
    self,
    rgb_hw3: UInt8[np.ndarray, "h w 3"],
    mask: UInt8[np.ndarray, "h w"],
) -> Image.Image:
    height: int = rgb_hw3.shape[0]
    width: int = rgb_hw3.shape[1]
    x: BFloat16[torch.Tensor, "batch channels latent_height latent_width"] = get_noise(...)
```

### Dataclass Type Annotations
Frame classes use extensive jaxtyping annotations:
```python
@dataclass
class Frame:
    H: int | None = None
    W: int | None = None
    rgb: np.ndarray | None = None
    dpt: np.ndarray | None = None
    inpaint: Bool[ndarray, "H W"] | None = None
    inpaint_wo_edge: Bool[ndarray, "H W"] | None = None
    intrinsic: Float[ndarray, "3 3"] | None = None
    cam_T_world: Float[ndarray, "4 4"] | None = None
```

References: [beartype docs](https://beartype.readthedocs.io/en/latest/), [jaxtyping docs](https://docs.kidger.site/jaxtyping/)

## Development Workflow

### Environment Setup
```bash
# Primary commands - use pixi for all package management
pixi install              # Install all dependencies
pixi run python <script>  # Run scripts in pixi environment
```

### Model Checkpoints
Models expect checkpoints in `./ckpt/` directory:
- `./ckpt/flux_fill/flux1-fill-dev.safetensors`
- `./ckpt/flux_fill/ae.safetensors` 

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rerun-io/vistadream](https://github.com/rerun-io/vistadream) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
