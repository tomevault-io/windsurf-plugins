---
trigger: always_on
description: This is a C implementation of two image synthesis model families:
---

This is a C implementation of two image synthesis model families:
- Flux.2 Klein (4B and 9B variants)
- Z-Image-Turbo (6B)

The project is called "Iris" (from the Greek goddess of the rainbow).
The Flux models are created by Black Forest Labs.
Z-Image-Turbo is published as `Tongyi-MAI/Z-Image-Turbo`.

Model type and architecture are autodetected from model metadata/config files.
Do not rely on hardcoded dimensions when a config value is available.

# Naming Convention

- **`iris_`** prefix for all shared/generic identifiers
- **`_flux`** postfix on internal functions/types specific to the Flux model family
- **`_zimage`** postfix on internal functions/types specific to Z-Image
- **No postfix** on public API functions (they route internally by model type)
- **Unchanged**: `qwen3_*`, `safetensors_*`, `zi_*` internal helpers (component-level namespaces)

# Supported Model Variants

Flux variants:
- **4B Distilled** (`flux-klein-4b`): 4 steps, no CFG, fast.
- **4B Base** (`flux-klein-4b-base`): 50 steps default, CFG, higher quality but much slower.
- **9B Distilled** (`flux-klein-9b`): 4 steps, larger model, higher quality. Non-commercial license.
- **9B Base** (`flux-klein-9b-base`): 50 steps default, CFG. Non-commercial license.

Z-Image variant:
- **Z-Image-Turbo** (`zimage-turbo`): distilled S3-DiT model, default 8 NFE (9 scheduler values), guidance 0.0.

# High-Level Capabilities

- Flux supports both txt2img and img2img with text conditioning.
- Z-Image currently supports txt2img path in this codebase.
- Text embeddings are generated via Qwen3.
- VAE is used for latent/image conversion in both families.

# File Structure

```
iris.c                    - Main library (model load, generation routing)
iris_transformer_flux.c   - Flux diffusion transformer (MMDiT)
iris_transformer_zimage.c - Z-Image transformer (S3-DiT)
iris_sample.c             - Sampling/denoising loops (Euler ODE)
iris_qwen3.c              - Qwen3 text encoder
iris_qwen3_tokenizer.c    - BPE tokenizer
iris_vae.c                - VAE encoder/decoder
iris_kernels.c            - CPU kernels (softmax, RMSNorm, etc.)
iris_metal.m              - Metal GPU acceleration runtime
iris_metal.h              - Metal/GPU API surface
iris_shaders.metal        - Metal compute kernels
iris_safetensors.c        - Weight loading
iris_image.c              - Image I/O (PNG/PPM/JPEG)
png.c                     - PNG encoder/decoder
jpeg.c                    - JPEG decoder
iris_cli.c                - Interactive CLI mode (REPL)
embcache.c                - Embedding cache (4-bit quantized)
linenoise.c               - Line editing library
terminals.c               - Terminal handling
main.c                    - CLI entry point
```

# Build Targets

This project implements three targets:
- MPS: Apple Silicon GPU path.
- BLAS: optimized CPU inference via BLAS/OpenBLAS.
- generic: pure C fallback, very slow.

# Development Rules

- No additional project dependencies. Acceptable external deps are BLAS/OpenBLAS and Metal/MPS from macOS.
- Reject tiny speed gains that add complexity; prefer substantial wins.
- Always test code modifications with `make test`.
- Once changes are validated, commit them.
- Never add or commit unrelated unstaged files.
- Keep code simple and understandable; leave no dead code.
- If you optimize one backend, verify others were not regressed.
- Stick to standard C; avoid compiler-specific tricks/pragmas unless strictly required.

# How To Run

Flux examples:

    ./iris -d flux-klein-4b -p "a cat and a dog playing" -o /tmp/test.png
    ./iris -d flux-klein-4b-base -p "a cat and a dog playing" -o /tmp/test.png
    ./iris -d flux-klein-9b -p "a cat and a dog playing" -o /tmp/test.png
    ./iris -d flux-klein-9b-base -p "a cat and a dog playing" -o /tmp/test.png

Z-Image example:

    ./iris -d zimage-turbo -p "a fish" -o /tmp/zimage.png

If model weights are missing, use the download script only after user approval.

# Python Reference Implementations

For parity checks/debugging:

Flux references:
- Python venv in `./flux_env/`
- Official Flux Python code in `./flux2/`

Z-Image references:
- `flux_env/lib/python3.12/site-packages/diffusers/models/transformers/transformer_z_image.py`
- `flux_env/lib/python3.12/site-packages/diffusers/pipelines/z_image/pipeline_z_image.py`
- `flux_env/lib/python3.12/site-packages/diffusers/schedulers/scheduling_flow_match_euler_discrete.py`
- `flux_env/lib/python3.12/site-packages/diffusers/models/autoencoders/autoencoder_kl.py`

Rules:
- Never add/commit `flux_env/` or `flux2/`.
- If missing, ask user before recreating/downloading.

# Debugging Notes

- Reusable debug scripts belong in `./debug`.
- One-off throwaway debugging tools should be created in `/tmp` and discarded.
- JPEG code has dedicated tests/tools in `./jpg_test`.

# Flux Pipeline Overview

```
1. Text Encoding:    prompt -> Qwen3 -> [512, text_dim] embeddings
2. Latent Init:      random noise [H/16, W/16, 128]
3. Denoising Loop:   double blocks -> single blocks -> final layer -> velocity
4. VAE Decode:       latents -> VAE decoder -> RGB image

img2img: VAE-encode reference images and pass as extra tokens (in-context conditioning).

Base CFG: run transformer twice per step (empty prompt + conditioned prompt):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antirez/iris.c](https://github.com/antirez/iris.c) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
