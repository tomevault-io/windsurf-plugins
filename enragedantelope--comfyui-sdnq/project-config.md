---
trigger: always_on
description: You are developing **ComfyUI-SDNQ**, a custom node pack that enables loading and using SDNQ (SD.Next Quantization) models in ComfyUI. This provides significant VRAM savings (50-75%) while maintaining quality, enabling users to run large models like FLUX and SD3.5 on consumer hardware.
---

# CLAUDE.md - ComfyUI SDNQ Node Pack Development Guide

## Project Overview

You are developing **ComfyUI-SDNQ**, a custom node pack that enables loading and using SDNQ (SD.Next Quantization) models in ComfyUI. This provides significant VRAM savings (50-75%) while maintaining quality, enabling users to run large models like FLUX and SD3.5 on consumer hardware.

**Key Principle**: This integration requires NO monkeypatching of ComfyUI. SDNQ integrates cleanly through the `diffusers` library API.

---

## Critical First Steps

1. **Create context.md** immediately after starting. Update it continuously with:
   - What you've completed
   - Current blockers or issues
   - Lessons learned
   - Next steps / TODOs

2. **Read this entire file** before writing any code

3. **Test incrementally** - get basic loading working before adding features

---

## Architecture Decisions (DO NOT DEVIATE)

### Integration Approach
```python
# SDNQ registers into diffusers via import side-effect
from sdnq import SDNQConfig  # This single import enables SDNQ loading
import diffusers

# Pre-quantized models load transparently
pipe = diffusers.FluxPipeline.from_pretrained(
    "Disty0/FLUX.1-dev-qint8",
    torch_dtype=torch.bfloat16
)
```

### Model Storage
- Store SDNQ models in: `ComfyUI/models/diffusers/sdnq/`
- Support `extra_model_paths.yaml` for custom paths
- Use huggingface_hub for downloads to standard HF cache

### Node API Compatibility
- **Primary**: V1 API (NODE_CLASS_MAPPINGS) for broad compatibility
- **Secondary**: V3 API schema for future-proofing
- Both APIs should be supported from the same node classes

---

## Repository Structure

```
ComfyUI-SDNQ/
├── __init__.py                    # Entry point with dual V1/V3 support
├── nodes/
│   ├── __init__.py                # Node exports
│   ├── loader.py                  # SDNQModelLoader node
│   ├── quantizer.py               # SDNQQuantizer node (optional, Phase 2)
│   └── catalog.py                 # SDNQModelCatalog node (optional)
├── core/
│   ├── __init__.py
│   ├── registry.py                # Model registry & catalog
│   ├── downloader.py              # HuggingFace Hub integration
│   ├── wrapper.py                 # ComfyUI type wrappers (MODEL, CLIP, VAE)
│   └── config.py                  # Configuration helpers
├── requirements.txt               # Python dependencies
├── install.py                     # ComfyUI Manager install hook
├── pyproject.toml                 # Modern packaging
├── LICENSE                        # Apache 2.0
├── README.md                      # User documentation
├── CREDITS.md                     # Attribution to Disty0
└── context.md                     # YOUR RUNNING NOTES (create this!)
```

---

## Implementation Order

### Phase 1: Minimum Viable Product

**Goal**: Load a pre-quantized SDNQ model and output ComfyUI-compatible types

1. **Create project structure**
   - All directories and placeholder files
   - requirements.txt with dependencies
   - Basic __init__.py with empty NODE_CLASS_MAPPINGS

2. **Implement core/wrapper.py**
   - Create wrappers to convert diffusers pipeline components to ComfyUI types
   - Focus on MODEL, CLIP, VAE outputs
   - Study existing ComfyUI diffusers loaders for patterns

3. **Implement nodes/loader.py - SDNQModelLoader**
   - Input: model path (local file selection)
   - Input: dtype selection (bfloat16, float16)
   - Input: device options
   - Output: MODEL, CLIP, VAE
   - Start with local loading only

4. **Test with real model**
   - Download Disty0/FLUX.1-dev-qint8 manually to test folder
   - Verify node appears in ComfyUI
   - Verify outputs connect to standard ComfyUI nodes

### Phase 2: HuggingFace Integration

5. **Implement core/registry.py**
   - Hardcoded catalog of known SDNQ models from Disty0 collection
   - Model metadata (type, quant level, size, etc.)
   - Detection of locally installed models

6. **Implement core/downloader.py**
   - Use `huggingface_hub` for downloads
   - Progress callbacks for UI
   - Caching and resume support

7. **Update nodes/loader.py**
   - Add dropdown for model catalog
   - Add HuggingFace repo ID input for custom models
   - Auto-download on first use

### Phase 3: Advanced Features (Optional)

8. **Quantization node**
   - Convert existing checkpoints to SDNQ format
   - Use sdnq.loader.save_sdnq_model()

9. **V3 API schemas**
   - Add comfy_entrypoint() function
   - Define IO schemas with type hints

---

## Key Code Patterns

### Basic Node Structure (V1 API)
```python
class SDNQModelLoader:
    @classmethod
    def INPUT_TYPES(cls):
        return {
            "required": {
                "model_path": ("STRING", {"default": ""}),
                "dtype": (["bfloat16", "float16", "float32"],),
            },
            "optional": {
                "use_quantized_matmul": ("BOOLEAN", {"default": True}),
            }
        }

    RETURN_TYPES = ("MODEL", "CLIP", "VAE")
    RETURN_NAMES = ("model", "clip", "vae")
    FUNCTION = "load_model"
    CATEGORY = "loaders/SDNQ"


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EnragedAntelope/comfyui-sdnq](https://github.com/EnragedAntelope/comfyui-sdnq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
