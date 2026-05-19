---
trigger: always_on
description: Generates 3D character models from each card for AR/VR use. Uses existing card images as input: background removal → 3D mesh generation → format conversion.
---

# Tarot Card Project

A tarot card layout, spread, and reader system. Differs from Rider-Waite by having more cards, compatible with both I-Ching balance model and Tarot action model.

## Tools & Stack

- **ComfyUI** — image and video generation workflows
- **n8n** — orchestration and automation
- **Perplexity** — research
- **TypeScript**

## Directory Structure

```
ComfyUI/           ComfyUI GUI workflow JSONs (Card_Dealer_*.json, Make_Images_For_Card.json, etc.)
n8n/               n8n orchestration workflow JSONs (Generate Dealer Animations.json, Generate Card Tarot Images.json, etc.)
data/              TarotSpreadsheet2.ods (card definitions)
Perplexity/        Research docs
```

## Data Paths

- Card images: `D:/data/cards/Standard/`
- Card back image: `D:/data/cards/Standard/IMAGE_Deck_Back.png`
- Dealer animations output: `D:/data/Dealer_Animations/`
- Tarot spreadsheet: `data/TarotSpreadsheet2.ods`

## Executables

| Tool | Path | Notes |
|------|------|-------|
| ffmpeg | `D:/ComfyUI_windows_portable/ffmpeg-8.0.1-full_build/bin/ffmpeg.exe` | Video concat, format conversion |
| n8n | `D:/n8n/` (run via `npm start`) | Orchestration; installed as npm dep in `D:/n8n/package.json` |
| Node.js | `C:/Program Files/nodejs/node.exe` | Runtime for n8n |
| Python | `C:/Users/justi/AppData/Local/Microsoft/WindowsApps/python.exe` | |
| ComfyUI | `D:/ComfyUI_windows_portable/ComfyUI/` | Image/video generation; API at `http://127.0.0.1:8188` |

## ComfyUI Installation

- **Path**: `D:/ComfyUI_windows_portable/ComfyUI/`
- **Hardware**: 6GB VRAM, 32GB system RAM
- **LoadImage input dir**: `D:/ComfyUI_windows_portable/ComfyUI/input/` (images must be copied here for LoadImage node)
- **API endpoint**: `http://127.0.0.1:8188/prompt` (POST), `http://127.0.0.1:8188/history/{prompt_id}` (GET)

### Custom Nodes Installed

| Node Pack | Status | Notes |
|-----------|--------|-------|
| ComfyUI-VideoHelperSuite | Working | VHS_VideoCombine for MP4 output |
| comfyui-rmbg | Working | BiRefNetRMBG for background removal (used by 3D pipeline) |
| comfyui-mixlab-nodes | Working | TripoSR nodes: `LoadTripoSRModel_`, `TripoSRSampler_`, `SaveTripoSRMesh` |
| FramePack-F1-T2V | Broken | No model loader produces FramePackMODEL type |
| FramePack-HY | No models | diffusers/ folder is empty, needs HunyuanVideo model download |

### Custom Nodes NOT Installed

- **AnimateDiff** (ComfyUI-AnimateDiff-Evolved) — do NOT generate workflows using AnimateDiff nodes

## Available Models (Wan 2.2 I2V)

All paths relative to `D:/ComfyUI_windows_portable/ComfyUI/`:

| Model | Path |
|-------|------|
| Wan 2.2 I2V high noise (14B fp8) | `models/diffusion_models/wan2.2_i2v_high_noise_14B_fp8_scaled.safetensors` |
| Wan 2.2 I2V low noise (14B fp8) | `models/diffusion_models/wan2.2_i2v_low_noise_14B_fp8_scaled.safetensors` |
| LightX2V 4-step LoRA (high noise) | `models/loras/wan2.2_i2v_lightx2v_4steps_lora_v1_high_noise.safetensors` |
| umt5_xxl text encoder (fp8) | `models/text_encoders/umt5_xxl_fp8_e4m3fn_scaled.safetensors` |
| CLIP-L | `models/text_encoders/clip_l.safetensors` |
| Wan 2.1 VAE | `models/vae/wan_2.1_vae.safetensors` |

## Critical Compatibility Notes

These were discovered through debugging — do not repeat these mistakes:

1. **AnimateDiff is NOT installed** — don't use AnimateDiff nodes in any workflow
2. **DualCLIPLoader does NOT support type "wan"** — valid types: sdxl, sd3, flux, hunyuan_video, hidream, hunyuan_image, hunyuan_video_15, kandinsky5, kandinsky5_image, ltxv, newbie
3. **CLIPLoader (single) DOES support type "wan"** — Wan only needs one text encoder (umt5_xxl), not two
4. **FramePack F1-T2V and FramePack HY are incompatible** — they use different types (FramePackMODEL vs FP_DIFFUSERS_PIPELINE), cannot interoperate
5. **Card back image filename**: `IMAGE_Deck_Back.png` (not `CardBack.png`)
6. **ComfyUI GUI JSON vs API JSON**: GUI format uses `nodes[]` and `links[]` arrays with numeric IDs; API format uses `prompt` object with string node IDs ("1", "2", ...) containing `class_type` and `inputs`
7. **`WanImageToVideo` does NOT accept `end_image`** — it only supports `start_image`. For start+end frame pinning (stop-motion interpolation), use **`WanFirstLastFrameToVideo`** instead. It accepts both `start_image` and `end_image` as optional inputs with the same interface, and works with the same Wan 2.2 I2V model.

## Dealer Animation Workflows (Wan 2.2 I2V)

Four animations: Cut, Fan, Merge, Rotate. All share the same node graph, differing only in prompts and output filenames.

### Node Chain

```
UNETLoader(1) → LoraLoaderModelOnly(8) → ModelSamplingSD3(9) → KSampler(10)
CLIPLoader(2) → CLIPTextEncode(4,5) → WanImageToVideo(7) → KSampler(10)
VAELoader(3) → WanImageToVideo(7) + VAEDecode(11)
LoadImage(6) → WanImageToVideo(7)
KSampler(10) → VAEDecode(11) → VHS_VideoCombine(12)
```

### Key Settings

- **KSampler**: steps=4, cfg=1.0, sampler=uni_pc_bh2, scheduler=sgm_uniform, denoise=1.0
- **WanImageToVideo**: width=480, height=832, length=81, batch_size=1
- **VHS_VideoCombine**: frame_rate=12, format=video/h264-mp4
- **ModelSamplingSD3**: shift=1.0 (flow matching)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [justice8096/TarotCardProject](https://github.com/justice8096/TarotCardProject) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
