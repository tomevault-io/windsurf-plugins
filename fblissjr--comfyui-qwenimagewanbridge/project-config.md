---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Code and Writing Style Guidelines

- **No emojis** in code, display names, or documentation
- Do **NOT** commit code to git or stage code for git without me explicitly asking and approving it
- Keep all naming and display text professional
- Avoid "Pure", "Enhanced", "Advanced", "Ultimate" type prefixes - use descriptive names instead
- Always avoid redundancy and unnecessary complexity. If you need to make a v2, there needs to be a compelling reason for it instead of simply modifying the code.
- Clean, simple node names that describe what they do
- Keep descriptions minimal and factual

## Organization

- `nodes/` - Production-ready nodes only
- `nodes/docs/` - Detailed node documentation (README, per-node docs, guides)
- `nodes/archive/` - Legacy and experimental nodes
- `example_workflows/` - Example JSON workflows with comprehensive notes
- `internal/` - Internal documentation and analysis

## Documentation

### User Docs
- `nodes/docs/README.md` - Documentation index
- `nodes/docs/z_image_intro.md` - Z-Image intro guide ("WTF is this?")
- `nodes/docs/z_image_encoder.md` - Z-Image encoder reference (extended template format)
- `nodes/docs/z_image_character_generation.md` - Character consistency guide
- `nodes/docs/QwenImageBatch.md` - Batch node documentation
- `nodes/docs/QwenVLTextEncoder.md` - Standard encoder documentation
- `nodes/docs/QwenTemplateBuilder.md` - Template builder documentation
- `nodes/docs/resolution_tradeoffs.md` - Comprehensive resolution and scaling guide
- `nodes/docs/wan21_vae_upscale2x_guide.md` - Wan2.1-VAE-upscale2x integration guide
- `nodes/docs/qwen_wan_bridge_guide.md` - Qwen-to-Wan Video bridge guide

### Developer Docs
- `internal/COMFYUI_JAVASCRIPT_PYTHON_DEVELOPER_GUIDE.md` - JS/Python integration patterns
- `internal/comfyui_js_python_integration.md` - Original integration research

## Project Overview

ComfyUI nodes for Qwen-Image-Edit model, enabling text-to-image generation and vision-based image editing using Qwen2.5-VL 7B. Bridges DiffSynth-Studio patterns with ComfyUI's node system.

**Key Features (v2.9.12):**
- **Token counting** - Debug output shows actual token count vs 512 reference limit (uses ComfyUI's bundled Qwen tokenizer)
- **Padding filter** - `filter_padding` parameter (default on) matches diffusers/DiffSynth reference implementations
- **Extended template format** - Templates can include `add_think_block`, `thinking_content`, `assistant_content` in YAML frontmatter
- **Structured prompt templates** - `json_structured`, `yaml_structured`, `markdown_structured` with pre-configured thinking
- **JSON key quote filtering** - `strip_key_quotes` toggle on Z-Image nodes prevents JSON keys appearing as text in images - [docs](nodes/docs/z_image_encoder.md)
- **HunyuanVideo 1.5 T2V** - Text-to-video with Qwen2.5-VL encoder (39 video templates)
- **File-based template system** - Templates in `nodes/templates/*.md` files (single source of truth)
- **Template Builder → Encoder** - Single `template_output` connection handles everything
- QwenImageBatch node (auto-detection, aspect preservation, double-scaling prevention) - [docs](nodes/docs/QwenImageBatch.md)
- multi_image_edit mode (DiffSynth `encode_prompt_edit_multi` alignment)
- Resolution scaling guide - [docs](nodes/docs/resolution_tradeoffs.md)
- 16-channel VAE latents (vs standard 4-channel)
- Vision token processing with multi-image support
- Token dropping (34 for T2I, 64 for I2E/multi/inpainting) - DiffSynth-compatible
- Mask-based inpainting with diffusers blending pattern (experimental, not fully tested)
- Native ComfyUI integration via `CLIPType.QWEN_IMAGE`

**Models:**
- Text/Vision Encoder: `Qwen/Qwen2.5-VL-7B-Instruct` → `models/text_encoders/`
- DiT Model: `qwen-image-edit-2509` (fp8 or Nunchaku quantized) → `models/diffusion_models/`
- VAE: `qwen_image_vae.safetensors` (16-channel, standard) → `models/vae/`
  - Alternative: `Wan2.1-VAE-upscale2x` (16-channel, 2x decoder upscaling) - [guide](nodes/docs/wan21_vae_upscale2x_guide.md)

## Implementation Status

### Core Features
- Text-to-image generation (QwenVLTextEncoder) - [docs](nodes/docs/QwenVLTextEncoder.md)
- Single/multi-image editing with vision tokens
- QwenImageBatch (aspect preservation, auto-detection, double-scaling prevention) - [docs](nodes/docs/QwenImageBatch.md)
- **File-based template system (9 templates)** - [docs](nodes/docs/QwenTemplateBuilder.md)
  - Templates: `default_t2i`, `default_edit`, `multi_image_edit`, `artistic`, `photorealistic`, `minimal_edit`, `technical`, `inpainting`, `raw`
  - Stored in `nodes/templates/*.md` with YAML frontmatter
  - JavaScript UI auto-fills `custom_system` field for editing
- Resolution scaling guide - [docs](nodes/docs/resolution_tradeoffs.md)
- 16-channel VAE support
- Multi-image "Picture X:" labeling (auto, 1-3 images optimal)
- Token dropping (34 for T2I, 64 for I2E/multi/inpainting)
- RoPE position embedding fix for batch processing

### Deprecated / Experimental / Research Only
- ZImageWanVAEDecode - Experimental node to decode Z-Image latents with Wan VAE (scaling correction, for testing only)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fblissjr/ComfyUI-QwenImageWanBridge](https://github.com/fblissjr/ComfyUI-QwenImageWanBridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
