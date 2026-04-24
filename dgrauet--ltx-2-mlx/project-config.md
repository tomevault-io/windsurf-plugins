---
trigger: always_on
description: Pure MLX port of [LTX-2](https://github.com/Lightricks/LTX-2/) (Lightricks) for Apple Silicon. Three-package monorepo mirroring the reference structure:
---

# CLAUDE.md — ltx-2-mlx

## Project Overview

Pure MLX port of [LTX-2](https://github.com/Lightricks/LTX-2/) (Lightricks) for Apple Silicon. Three-package monorepo mirroring the reference structure:

- **ltx-core-mlx** (`ltx_core_mlx`) — model library: DiT, VAE, audio, text encoder, conditioning
- **ltx-pipelines-mlx** (`ltx_pipelines_mlx`) — generation pipelines: T2V, I2V, retake, extend, keyframe, IC-LoRA, two-stage
- **ltx-trainer** (`ltx_trainer_mlx`) - ltx-2 training, democratized.

Loads pre-converted MLX weights from the [LTX-2.3 MLX collection on HuggingFace](https://huggingface.co/collections/dgrauet/ltx-23). Weight conversion is handled by [mlx-forge](https://github.com/dgrauet/mlx-forge).

---

## Tech Stack

- Python 3.11+, `uv` workspace (monorepo with `packages/*`)
- MLX (`mlx>=0.31.0`) — Apple Silicon ML framework (unified CPU/GPU memory)
- `mlx-lm>=0.31.0` — for Gemma 3 text encoder loading
- `safetensors`, `huggingface-hub`, `numpy`
- Linter/formatter: ruff

---

## Architecture

```
packages/
├── ltx-core-mlx/                          # ltx_core_mlx
│   └── src/ltx_core_mlx/
│       ├── components/                    # Shared pipeline components
│       │   ├── guiders.py                 # Guidance strategies
│       │   └── patchifiers.py             # VideoLatentPatchifier, AudioPatchifier
│       │
│       ├── conditioning/                  # Latent conditioning system
│       │   ├── mask_utils.py              # build/update/resolve attention masks
│       │   └── types/
│       │       ├── attention_strength_wrapper.py # Attention strength wrapping
│       │       ├── latent_cond.py         # LatentState, VideoConditionByLatentIndex
│       │       ├── keyframe_cond.py       # VideoConditionByKeyframeIndex
│       │       └── reference_video_cond.py # VideoConditionByReferenceLatent (IC-LoRA)
│       │
│       ├── guidance/                      # Guidance utilities
│       │   └── perturbations.py           # Noise perturbation strategies
│       │
│       ├── loader/                        # Weight loading & LoRA fusion
│       │   ├── fuse_loras.py              # LoRA weight fusion
│       │   ├── primitives.py              # Loading primitives
│       │   ├── sd_ops.py                  # Safetensors loading operations
│       │   └── sft_loader.py              # Split safetensors loader
│       │
│       ├── model/
│       │   ├── audio_vae/                 # Audio VAE + vocoder + BWE
│       │   │   ├── audio_vae.py           # AudioVAEDecoder, AudioResBlock, AudioAttnBlock
│       │   │   ├── encoder.py             # AudioVAEEncoder
│       │   │   ├── vocoder.py             # BigVGANVocoder, SnakeBeta, Activation1d
│       │   │   ├── bwe.py                 # VocoderWithBWE, HannSincResampler, MelSTFT
│       │   │   └── processor.py           # AudioProcessor (STFT + mel filterbank)
│       │   │
│       │   ├── transformer/               # Diffusion Transformer (DiT)
│       │   │   ├── model.py               # LTXModel, X0Model, LTXModelConfig
│       │   │   ├── transformer.py         # BasicAVTransformerBlock (joint audio+video)
│       │   │   ├── attention.py           # Multi-head attention + RoPE + per-head gating
│       │   │   ├── feed_forward.py        # Gated MLP blocks
│       │   │   ├── rope.py                # Rotary position embeddings (SPLIT type)
│       │   │   ├── adaln.py               # AdaLayerNormSingle (9-param)
│       │   │   └── timestep_embedding.py  # Sinusoidal + MLP timestep encoding
│       │   │
│       │   ├── upsampler/                 # Neural latent upscaler
│       │   │   └── model.py               # LatentUpsampler, SpatialRationalResampler
│       │   │
│       │   └── video_vae/                 # Video VAE
│       │       ├── video_vae.py           # VideoDecoder (streaming), VideoEncoder
│       │       ├── convolution.py         # Conv3dBlock (causal + reflect padding)
│       │       ├── resnet.py              # ResBlock3d, ResBlockStage
│       │       ├── sampling.py            # DepthToSpaceUpsample, pixel_shuffle_3d
│       │       ├── tiling.py              # Tiled VAE encoding/decoding
│       │       ├── normalization.py       # pixel_norm (RMS)
│       │       └── ops.py                 # PerChannelStatistics
│       │
│       ├── text_encoders/                 # Text encoding (Gemma 3)
│       │   └── gemma/
│       │       ├── embeddings_connector.py  # Embeddings1DConnector (RoPE + registers)
│       │       ├── feature_extractor.py     # GemmaFeaturesExtractorV2 (video/audio projections)
│       │       └── encoders/
│       │           ├── base_encoder.py      # Gemma 3 12B wrapper via mlx-lm
│       │           └── prompts/             # System prompt templates
│       │               ├── gemma_t2v_system_prompt.txt
│       │               └── gemma_i2v_system_prompt.txt
│       │
│       └── utils/
│           ├── positions.py   # compute_video_positions, compute_audio_positions
│           ├── weights.py     # load_split_safetensors, apply_quantization
│           ├── memory.py      # aggressive_cleanup, get_memory_stats
│           ├── image.py       # prepare_image_for_encoding

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dgrauet) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
