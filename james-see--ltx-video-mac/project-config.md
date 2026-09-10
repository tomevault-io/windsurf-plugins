---
trigger: always_on
description: Native SwiftUI macOS app (14+). Generation is a local subprocess — not in-process MLX. You are talking to a 20+ year full-stack owner: be concise, no tutorial filler.
---

# Agent notes — ltx-video-mac

Native SwiftUI macOS app (14+). Generation is a local subprocess — not in-process MLX. You are talking to a 20+ year full-stack owner: be concise, no tutorial filler.

Three backends via `GenerationBackend` on `LTXModel`:

| Backend | When | Process |
|---|---|---|
| `mlxVideoWithAudio` | LTX-2 / 2.3 `notapalindrome` packs | `python -m mlx_video.generate_av` |
| `ltx2Mlx` | LTX-2.5 + LTX-2.3 12GB | `ltx-2-mlx generate` (or adapter wrapper) |
| `h3c` | MiniMax H3 | native `./h3` (C/Metal) |

## Repos

| Repo | Typical path | Role |
|---|---|---|
| This app | `/Users/jc/p/ltx-video-mac` (or clone anywhere) | SwiftUI shell, queue, UI, local REST API |
| Library | `/Users/jc/p/mlx-video-with-audio` | LTX-2 / 2.3 I2V/T2V + audio (PyPI `mlx-video-with-audio`) |
| LTX-2.5 runtime | `~/projects/ltx-2-mlx` (Preferences local toggle) | `dgrauet/ltx-2-mlx` — do **not** port 2.5 into mlx-video-with-audio |
| H3 engine | `~/projects/h3.c` or App Support clone | `antirez/h3.c` (BF16/Turbo); int8 fork for `minimax_h3_int8` |

**Preference hardcodes:** local overrides look at `~/projects/mlx-video-with-audio` and `~/projects/ltx-2-mlx` regardless of where this git checkout lives. Symlink or clone there when using the toggles.

`LTXBridge` prefers pip `mlx-video-with-audio` unless `~/projects/mlx-video-with-audio` is newer, Preferences “Use local mlx-video-with-audio repo” is on, or `LTX_FORCE_LOCAL_MLX_VIDEO=1`.

If a PR changes the Python CLI (`--keyframe`, kwargs on `generate_video_with_audio`, etc.), land and **publish the library first**. Shipping the app against an unreleased flag breaks I2V for everyone on PyPI.

## LTX-2.5 reference

Upstream: [Lightricks/LTX-2.5](https://huggingface.co/Lightricks/LTX-2.5) (CUDA `ltx-pipelines` / Diffusers / Comfy). Mac path: app pack [notapalindrome/ltx25-mlx](https://huggingface.co/notapalindrome/ltx25-mlx) (MLX conversion provenance: [mlx-community/ltx-2.5-mlx](https://huggingface.co/mlx-community/ltx-2.5-mlx)) + runtime [dgrauet/ltx-2-mlx](https://github.com/dgrauet/ltx-2-mlx) `@v0.15.2`. Plan: `plans/ltx-2.5-support.md`.

### Catalog ↔ official model family

| App `model_id` | Pack | CLI shape | Maps to official |
|---|---|---|---|
| `ltx25_distilled` | `notapalindrome/ltx25-mlx` (~110GB w/ LoRA) | `--distilled` | Distilled DiT, fixed **8 steps, CFG=1** |
| `ltx25_dev` | same (LoRA bundled) | `--two-stage` + `transformer-dev` + fuse `ltx-2.5-22b-distilled-lora-450-bf16` | Dev two-stage (half-res + spatial upscale + distilled-LoRA refine); stage-1 steps/CFG from UI (defaults 30 / 3), stage-2 = 3 |
| `ltx25_distilled_ditq8` | same + `notapalindrome/ltx25-mlx-ditq8` overlay | `--distilled` on shadowed pack | Distilled + Q8 DiT (0.15.2 has no `--dit`) |

Gemma 4 is **bundled** in `gemma4-12b-ltx-v1/`. Do not use the Gemma 3 picker for 2.5. `mlx-community/ltx-2.5-mlx-q8` is the **text encoder**, not a DiT quant — never catalog it as DiT.

App reuses a complete local `mlx-community/ltx-2.5-mlx` (or `-ditq8`) cache for the notapalindrome catalog ids — same conversion, skip re-download. Dev LoRA similarly reuses a cached `dgrauet/ltx-2.5-mlx` file when the pack copy is absent.

App loads the community pack through bundled `LTXVideoGenerator/Resources/ltx25_community_adapter.py` (patches Gemma 4 path + mixed-precision quant + DurationHead skip). Keep that file in the Xcode Resources target.

### Implemented vs official “what's new”

| Official feature | Status here |
|---|---|
| Distilled 8-step / CFG=1 | Yes |
| Dev + distilled LoRA two-stage | Yes (`ltx25_dev`; LoRA bundled in `notapalindrome/ltx25-mlx`) |
| Gemma 4 12B TE | Yes (pack + adapter) |
| Prompt enhancer | Yes → `--enhance-prompt` (not Gemma 3 preview path) |
| Audio VAE + vocoder | Yes (in pack) |
| DiffVAE + spatial/temporal upscalers | In pack; two-stage uses spatial upscaler |
| I2V first-frame | Yes → `--image` |
| `num_frames % 8 == 1`; W/H ÷32 | Defaults OK; app snaps W/H to **64** (stricter) |
| Extra timeline keyframes | Yes — repeatable `--image PATH FRAME_IDX STRENGTH` (pixel idx; mlx-video uses latent `--keyframe`) |
| Duration predictor (omit frames) | Load fixed in adapter for community split q/k/v; app still passes `-f` from slider ([upstream #125](https://github.com/dgrauet/ltx-2-mlx/issues/125)) |
| Native multishot / DFR | **Not wired** — no CLI in 0.15.2 ([upstream #127](https://github.com/dgrauet/ltx-2-mlx/issues/127)). `--segment` Prompt Relay not in UI yet |
| Disable audio | **Ignored** on 0.15.2 ([upstream #126](https://github.com/dgrauet/ltx-2-mlx/issues/126)) |

Parity epic: https://github.com/james-see/ltx-video-mac/issues/85

**Prompting:** official LTX-2.5 style only — [ltx.io](https://ltx.io/blog/ltx-2-5-prompt-guide) / [docs.ltx.io](https://docs.ltx.io/api-documentation/implementation-guides/prompting-guide). App copy: `EXAMPLES.md`, `docs/usage.md`, README tips. Flowing present-tense paragraphs, quoted dialogue, prose hard cuts. No `START FRAME` / JUMP CUT lists. Prefer single continuous take on Mac Distilled/Dev until multishot/DFR is wired.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [james-see/ltx-video-mac](https://github.com/james-see/ltx-video-mac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
