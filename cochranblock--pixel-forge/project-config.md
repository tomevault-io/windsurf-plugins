---
trigger: always_on
description: Part of the Cochran Block ecosystem. Powered by KOVA. Human direction, AI execution.
---

# pixel-forge (pixel art generator)

## Identity

Part of the Cochran Block ecosystem. Powered by KOVA. Human direction, AI execution.

## Build Commands

| Action | Command |
|--------|---------|
| Build (Metal) | `cargo build --release -p pixel-forge` |
| Build (CUDA) | `cargo build --release -p pixel-forge --features cuda --no-default-features` |
| Build (CPU) | `cargo build --release -p pixel-forge --no-default-features` |
| Train Anvil | `cargo run --release -- train --data data_v3_32 --anvil --epochs 200 --lr 2e-4 --batch-size 16 --no-ema --checkpoint-every 1` |
| Train Quench | `cargo run --release -- train --data data_v3_32 --medium --epochs 200 --lr 2e-4 --batch-size 16 --no-ema --checkpoint-every 1` |
| Train Cinder | `cargo run --release -- train --data data_v3_32 --epochs 500 --lr 2e-4 --batch-size 128 --no-ema` |
| Train Experts | `cargo run --release -- train-experts --data data --epochs 50` |
| Train Judge | `cargo run --release -- train-judge` |
| Train PaletteNet | `cargo run --release -- train-palette-net --data data_v3_32 --output models/palette.safetensors` |
| Prep Silo Cond | `cargo run --release -- prep-silo-cond --data data_v3_32 --output data_silo_cond_32` |
| Train Cinder-detail | `cargo run --release -- train --data data_v3_32 --condition data_silo_cond_32 --epochs 100 --lr 1e-4 --resume pixel-forge-cinder.safetensors --output models/cinder-detail.safetensors` |
| Generate (Anvil) | `cargo run --release -- anvil character --count 4 --steps 40 --palette stardew` |
| Generate (Cinder) | `cargo run --release -- generate character --palette stardew` |
| Tiered Pipeline | `cargo run --release -- tiered warrior --count 4 --palette endesga32` |
| Cascade (MoE) | `cargo run --release -- cascade character --count 16` |
| Auto-detect | `cargo run --release -- auto character` |
| Scene | `cargo run --release -- scene biome` |
| GUI | `cargo run --release` |
| Plugin (kova) | `cargo run --release -- plugin` |

## Architecture

| Module | File | Purpose |
|--------|------|---------|
| tiny_unet | src/tiny_unet.rs | Cinder model — 1.09M params, [32,64,64] channels |
| medium_unet | src/medium_unet.rs | Quench model — 5.83M params, [64,128,128] + self-attention |
| anvil_unet | src/anvil_unet.rs | Anvil model — 16.9M params, XL |
| train | src/train.rs | Training loop, sampling, data pipeline |
| app | src/app.rs | egui GUI — device auto-detect, generation, gallery |
| device_cap | src/device_cap.rs | Device detection, tier selection, benchmarks |
| cluster | src/cluster.rs | Distributed generation across SSH nodes |
| combiner | src/combiner.rs | SlotGridTransformer — scene composition |
| judge | src/judge.rs | MicroClassifier — quality scoring |
| expert | src/expert.rs | MoE expert heads — shape/color/detail/class |
| expert_train | src/expert_train.rs | Expert training on frozen Quench |
| moe | src/moe.rs | Cascade pipeline — Cinder → Quench + Experts |
| scene | src/scene.rs | 8x8 SceneGrid, biome generation |
| swipe_store | src/swipe_store.rs | Tinder-style swipe data for judge training |
| lora | src/lora.rs | Rank-4 LoRA adapters for TinyUNet |
| discriminator | src/discriminator.rs | Quality gate — binary classifier |
| palette | src/palette.rs | Color palettes + quantization |
| class_cond | src/class_cond.rs | Hybrid conditioning: 10 super-categories + 12 binary tags |
| plugin | src/plugin.rs | JSON protocol for kova integration |
| poa | src/poa.rs | Proof of Authorship signing |
| pipeline | src/pipeline.rs | SD pipeline (optional, desktop) |
| gpu_lock | src/gpu_lock.rs | File-based GPU lock for training |
| relight | src/relight.rs | 4-directional sprites from SDF + normals |
| quantize | src/quantize.rs | f32 → f16 model quantization |
| nanosign | src/nanosign.rs | NanoSign BLAKE3 model integrity — sign on save, verify on load |
| tiered_pipeline | src/tiered_pipeline.rs | Tiered pipeline — silo router → noise blend → Cinder detail |
| palette_net | src/palette_net.rs | PaletteNet — ~100K MLP predicting class palette from conditioning |

## Model Tiers

| Tier | Name | Token | Params | Size | File |
|------|------|-------|--------|------|------|
| Tiny | Cinder | m0 | 1.09M | 4.2MB | pixel-forge-cinder.safetensors |
| Medium | Quench | m1 | 5.83M | 22MB | pixel-forge-quench.safetensors |
| XL | Anvil | m2 | 16.9M | 64MB | pixel-forge-anvil.safetensors |

## Tiered Pipeline

Three specialist models chained for higher quality output:

```
class_cond::lookup(class)
    ↓
[Stage 1: Shape]  Per-class MicroUNet silo (~97K params)
    → coarse RGB sprite → blend with noise at refine_from_t=0.45
    → falls back to pure noise if no silo available
    ↓
[Stage 2: Palette]  PaletteNet (~100K params)   [Phase 2 — train first]
    → predict 8 class-appropriate palette colors
    → quantize to nearest endesga32 entries
    ↓
[Stage 3: Detail]  Cinder (1.09M params, or Cinder-detail 6ch)
    → DDIM from silo-seeded start → full-resolution sprite
    ↓
palette::quantize() → final output
```

**Training sequence for best results:**
1. `train-palette-net` — 10 min CPU (extracts K-means colors per class)
2. `prep-silo-cond --data data_v3_32 --output data_silo_cond_32` — coarsen training images (32→16→32) as conditioning hints

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cochranblock/pixel-forge](https://github.com/cochranblock/pixel-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
