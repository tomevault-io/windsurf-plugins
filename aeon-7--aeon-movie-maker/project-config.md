---
trigger: always_on
description: >
---


# Movie Maker Fast — LTX 2.3 cinematic video pipeline

> 📖 **For execution**, read [AGENTS.md](AGENTS.md) first — it has the
> glossary, decision tree, literal copy-pasteable recipes, and a
> troubleshooting table optimized for AI agents. **This file (SKILL.md)
> is the deep-dive reference** for prompt-engineering recipes,
> chunking strategy details, and advanced configuration. Use SKILL.md
> when you need to understand *why* something works the way it does;
> use AGENTS.md when you just need to *do* the thing.

> Companion to `radio-drama-production` (audio only), `music-producer` (standalone music), and `tts-voice-designer` (voice casting). This skill is the **video engine**; it imports all three of those for the audio passes.

## 0. Target host + tool

- **Host:** `${SSH_USER}@127.0.0.1` (Workstation — RTX 5090, 64 GB RAM)
- **Tool:** `${COMFYUI_ROOT}\scene_production_tool\movie_maker_fast.py`
- **Companion tools** (invoked by this one for audio):
  - `scene_production_tool/radio_drama.py` — dialogue TTS + SFX priority chain
  - `music_tool/music_maker.py` — music cues via ACE Step XL base + APG chain
- **ComfyUI endpoint:** `http://127.0.0.1:8188`

## 1. Why this skill exists (and when NOT to use it)

The original cinema pipeline (`AGENT_CINEMA_AUTOPILOT` using `render_all_acts.py` + WAN 2.1 MultiTalk) produces **very tight lip-synced dialogue** but takes ~20–30 min per shot. For a 10-minute drama that's 4–6 hours of render.

Movie Maker Fast uses **LTX 2.3 distilled fp8** — a video-only model tuned for speed. A 7-second clip at 832×480 renders in ~75 s warm on the 5090. The full 10-minute drama renders in ~30–40 min. **~10–15× speedup.**

**Use this skill when:**
- Visuals are the primary deliverable; lip-sync is "close enough"
- You want a cinematic film with musical scoring + SFX, dialogue may be VO or off-frame
- Speed matters (previews, iterations, multi-shot drafts before committing)
- The production has many scenes (>15) where MultiTalk's per-shot cost is prohibitive

**Use `AGENT_CINEMA_AUTOPILOT` (slow WAN) instead when:**
- On-screen character dialogue requires tight lip-sync (every word matches mouth)
- Hero shots where motion naturalness on the speaker is paramount
- Short-form work where the 20-min-per-shot cost is acceptable

Both pipelines can coexist — the same `screenplay.json` works for both.

## 2. Three render modes — `--mode fast | quality | abstract`

LTX 2.3 is trained predominantly on real-world video. Each mode tunes the LoRA stack + sampler for a different content class. Pick by **what kind of video you're making**:

| Mode | Content class | Stack | Sampler | CFG | Steps |
|---|---|---|---|---|---|
| **`fast`** (default) | Narrative / character / real-world scenes | Distilled + IC-union + VBVR physics | euler | 3.0 | 20 |
| `quality` | Higher prompt-fidelity / motion variety | Non-distilled FP8 + distill LoRA @ 0.5 + IC-union + VBVR | euler | 3.0 | 30 |
| **`abstract`** | Fractals, geometry, artwork in motion, psychedelic, non-physical | NO always-on LoRAs (physics would hurt) | **euler_ancestral** | **5.0** | **30** |

Why abstract drops the physics + reference LoRAs:
- **VBVR** enforces object permanence, gravity, and collision realism — exactly wrong for a pulsing mandala or fractal unfold.
- **IC-LoRA union control** carries reference-scene semantics that don't apply to non-representational content.
- **euler_ancestral** adds stochastic variation each step, which morphs abstract content more expressively than plain euler.
- **Higher CFG (5 vs 3) + 30 steps** compensate for the distilled model's natural-video bias when asked for unfamiliar geometry.

## 2a. Model stack (all on disk, all verified)

### Fast mode (DEFAULT — `--mode fast`)

| Slot | File | Role |
|---|---|---|
| Base | `ltx-2.3-22b-distilled-fp8.safetensors` (27 GB) | Video-only distilled 22B, fp8 |
| Video VAE | `LTX23_video_vae_bf16.safetensors` | |
| Text encoder | `gemma_3_12B_it.safetensors` | Base Gemma-3 12B IT (Comfy-Org/ltx-2 split) |
| Abliteration LoRA | `gemma-3-12b-it-abliterated_heretic_lora_rank64_bf16.safetensors` | Available on disk; not auto-applied (needs CLIP-side wiring — manual workflow only) |
| LoRA (always) | `ltx-2.3-22b-ic-lora-union-control-ref0.5.safetensors` @ 1.0 | Reference-based char/scene control |
| LoRA (always) | `ltx2/Ltx2.3-Licon-VBVR-I2V-96000-R32.safetensors` @ 1.0 | Physics / object permanence |

**No distilled-lora-384 in fast mode** — already baked into the checkpoint. Adding it would over-distill.

### Quality mode (`--mode quality`)

| Slot | File | Role |
|---|---|---|
| Base | `ltx-2.3-22b-dev-fp8.safetensors` (~29 GB) | Non-distilled FP8 base — higher prompt-fidelity, more motion variety |
| Video VAE | same | |
| Text encoder | same | |
| LoRA | `ltx-2.3-22b-distilled-lora-384.safetensors` @ 0.5 | Partial distill — compresses step count without baking in full distilled behaviour (root of `loras/` — no `ltx2/` prefix) |
| LoRA | `ltx-2.3-22b-ic-lora-union-control-ref0.5.safetensors` @ 1.0 | |
| LoRA | `ltx2/Ltx2.3-Licon-VBVR-I2V-96000-R32.safetensors` @ 1.0 | |

Quality mode is ~30–50% slower than fast mode. Use it when fast-mode output looks too "average" or when you need stronger prompt adherence. No joint-AV path — audio comes exclusively from the separate audio stack (Qwen3-TTS / ACE-Step / MMAudio).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AEON-7/aeon-movie-maker](https://github.com/AEON-7/aeon-movie-maker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
