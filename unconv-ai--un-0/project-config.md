---
trigger: always_on
description: See [README.md](README.md) for what this project is, the model recipe, and the
---

# Un0

See [README.md](README.md) for what this project is, the model recipe, and the
full setup / training / inference / evaluation commands. That is the source of
truth for both readers and contributors.

## Quick reference

```bash
uv sync --group dev    # core + tests + ruff
uv run pytest          # unit tests
uv run ruff check      # lint
```

## Hardware notes

- On Blackwell (sm_100+; B300 reports sm_103), the cuDNN 9.x bundled with
  `torch 2.11+cu128` has no valid SDPA execution plan, so the compiled DINO
  attention crashes with `No valid execution plans built`. Both training entry
  points call `common.disable_cudnn_sdp_on_blackwell()`, which falls back to
  flash, gated on compute capability `>= 10` so pre-Blackwell GPUs (H200, A100)
  keep cuDNN attention. After changing SDPA backends, clear the Inductor cache
  (`/tmp/torchinductor_*`) so a stale compiled graph doesn't keep calling the
  cuDNN attention op.

## Reproducing the released checkpoints: config truth

The released checkpoints (on the Hugging Face Hub) were trained with the configs
below. **These values are fixed** — reproduce against these tables, and note
where a value is a `build_*_model()` default or a module constant rather than a
CLI flag (so `--help` alone does not tell the whole story). CIFAR-10 and
ImageNet-64 are separate recipes and differ in several places (precision, weight
decay, queue size, parameterization, relativization, per-size LR).

### CIFAR-10 (`cifar10/n1024`, `n2048`, `n4096`)

`train_cifar10.py`'s **CLI defaults** reproduce `n2048`/`n4096`; the only knob to
change for `n1024` is `--lr`. Note the arch knobs (`num_steps`, `solver`,
`relativization`) are CLI args whose *defaults* differ from the
`build_cifar10_model()` builder defaults — the CLI default is the checkpoint
value.

| setting | n1024 | n2048 | n4096 | source |
| --- | --- | --- | --- | --- |
| `n_oscillators` | 1024 | 2048 | 4096 | `build_cifar10_model()` |
| params | 1.3M | 4.9M | 19.4M | (derived) |
| **`lr` (peak)** | **2.683e-3** | **1.389e-3** | **1.389e-3** | `--lr` |
| `batch_size` | 2048 | 2048 | 2048 | `--batch-size` |
| `epochs` | 1200 | 1200 | 1200 | `--epochs` |
| `precision` | fp32 | fp32 | fp32 | `--precision` (CLI default bf16; see note) |
| `weight_decay` | 1e-3 | 1e-3 | 1e-3 | `WEIGHT_DECAY` |
| `dino_weight` / `pixel_weight` | 1.0 / 0.004 | — | — | `--dino-weight` / `--pixel-weight` |
| `queue_size` / `num_pos` | 2048 / 64 | — | — | `--queue-size` / `--num-pos` |
| `queue_storage_dtype` | float32 | — | — | `--queue-storage-dtype` |
| `num_steps` | 10 | 10 | 10 | `--num-steps` (builder default 25) |
| `solver` | euler | — | — | `--solver` (builder default rk4) |
| `relativization` | mean_relative | — | — | `--relativization` (builder default ref_oscillator) |
| `parameterization` | standard | — | — | `--parameterization` |
| `n_conditional_oscillators` | 8 | 8 | 8 | `build_cifar10_model()` |
| `class_dropout_prob` | 0.1 | — | — | `build_cifar10_model()` |
| `betas` | (0.9, 0.95) | — | — | `BETA1/BETA2` |
| warmup / decay | 0.1 dur, linear→0 | — | — | `WARMUP_FRACTION` |
| `grad_clip` | 1.0 | — | — | `GRAD_CLIP_NORM` |
| `gamma` | 0.2 | — | — | `GAMMA` |
| seed | 42 | 42 | 42 | `--seed` |

### ImageNet-64 (`imagenet64/n6656`, `n10240`, `n16384`)

| setting | n6656 | n10240 | n16384 | source |
| --- | --- | --- | --- | --- |
| `n_oscillators` | 6656 | 10240 | 16384 | `build_imagenet64_model()` |
| params | 57M | 130M | 322M | (derived) |
| **`lr` (peak)** | **1.585e-3** | **1e-3** | **1e-3** | `--lr` |
| `batch_size` (per device) | 2048 | 2048 | 2048 | `--batch-size` |
| global batch (× 8 GPUs) | 16384 | 16384 | 16384 | — |
| `epochs` / `lr_schedule_epochs` | 3600 | 3600 | 3600 | `--epochs` / `--lr-schedule-epochs` |
| `precision` | bf16 | bf16 | bf16 | `--precision` |
| `weight_decay` | 0.0 | 0.0 | 0.0 | `WEIGHT_DECAY` |
| `dino_weight` / `pixel_weight` | 1.0 / 0.1 | — | — | `--dino-weight` / `--pixel-weight` |
| `queue_size` / `num_pos` | 128 / 64 | — | — | `--queue-size` / `--num-pos` |
| `queue_storage_dtype` | bfloat16 | — | — | `--queue-storage-dtype` |
| `num_steps` (euler) | 10 | 10 | 10 | `build_imagenet64_model()` |
| `n_conditional_oscillators` | 1 | 1 | 1 | `build_imagenet64_model()` |
| `parameterization` | mup | mup | mup | `build_imagenet64_model()` |
| `relativization` | ref_oscillator | — | — | `build_imagenet64_model()` |
| `class_dropout_prob` | 0.1 | — | — | `build_imagenet64_model()` |
| `betas` | (0.9, 0.95) | — | — | `BETA1/BETA2` |
| warmup / decay | 0.15 dur, linear→0 | — | — | `WARMUP_FRACTION` |
| `grad_clip` | 2.0 | — | — | `GRAD_CLIP_NORM` |
| `gamma` / `num_classes_per_step` | 0.2 / 64 | — | — | `GAMMA`, `NUM_CLASSES_PER_STEP` |
| drift temperatures | (0.02, 0.05, 0.2) | — | — | `DRIFT_TEMPERATURES` (`losses.py`) |
| seed | 42 | 42 | 42 | `--seed` |

In both tables `—` means identical to the leftmost (smallest) size. **The
smallest model used a higher LR** in each family (CIFAR `n1024` = 2.683e-3,
ImageNet `n6656` = 1.585e-3, from their own LR sweeps); the larger sizes share
one LR.

Two things these tables make explicit:

- **Many values are not CLI args** (or their CLI default differs from the model
  builder's). Architecture and training constants live in `build_*_model()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unconv-ai/Un-0](https://github.com/unconv-ai/Un-0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
