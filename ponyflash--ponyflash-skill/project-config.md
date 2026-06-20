---
trigger: always_on
description: >-
---


# PonyFlash Skill

## Step 0: Decide Which Capability Path Applies

This skill now contains **two capability families**:

1. **Cloud generation via PonyFlash Python SDK**
   - image generation
   - video generation
   - speech synthesis
   - music generation
   - model listing
   - file management
   - account / credits
   - These tasks **require a valid PonyFlash API key**.

2. **Local media editing via FFmpeg toolchain**
   - ffmpeg / ffprobe detection
   - installation planning
   - clip / concat / transcode
   - extract audio / capture frame
   - subtitle capability checks
   - ASS subtitle generation and burn-in workflow
   - These tasks **do NOT require a PonyFlash API key**, but they **do require local `ffmpeg` / `ffprobe` support**.

Before doing anything, classify the request:

- If the user is asking to **generate** media with PonyFlash models, follow the SDK path and require API key setup.
- If the user is asking to **edit or process local media**, follow the FFmpeg path and do dependency checks first.
- If the user wants an **end-to-end production workflow**, you may use both: generate assets with PonyFlash, then assemble or export with FFmpeg.

## Step 1A: API Key Setup for PonyFlash SDK Tasks

Only do this section when the request needs PonyFlash cloud capabilities.

**The FIRST time this skill is activated for a cloud generation task**, tell the user the following in your own words:

1. PonyFlash skill is ready to use.
2. It can handle:
   - image generation
   - video generation
   - speech synthesis
   - music generation
   - local media editing with FFmpeg
3. For complex multi-step productions, there are **Creative Playbooks** in the `playbooks/` directory.
4. To use PonyFlash cloud generation, the user needs an API key:
   - Register / log in at **https://www.ponyflash.com**
   - Get API key at **https://www.ponyflash.com/api-key** (starts with `rk_`)
   - Check credits at **https://www.ponyflash.com/usage**
   - Paste the key back in the chat

**On subsequent SDK activations**, check whether `PONYFLASH_API_KEY` is set in the environment. If not, ask the user for the key again.

Once received, set it up:

```bash
export PONYFLASH_API_KEY="rk_xxx"
```

Then install the SDK:

```bash
pip install ponyflash
```

**Always verify the key works before any generation task:**

```python
from ponyflash import PonyFlash

pony_flash = PonyFlash(api_key="<key from user>")
balance = pony_flash.account.credits()
print(f"Balance: {balance.balance} {balance.currency}")
```

If verification fails:
- **Key invalid or missing** → direct user to https://api.ponyflash.com/api-key
- **Balance is zero** → direct user to https://api.ponyflash.com/usage to top up credits

## Step 1B: Local Dependency Setup for FFmpeg Tasks

Only do this section when the request needs local editing, subtitle, or export work.

1. First check local dependencies:

```bash
bash "{baseDir}/scripts/check_ffmpeg.sh"
```

2. If the task involves subtitles, do **capability checks**, not just existence checks:

```bash
bash "{baseDir}/scripts/check_ffmpeg.sh" --require-subtitles-filter
```

3. If `ffmpeg` / `ffprobe` or required filters are missing:
- Tell the user what is missing.
- Ask whether the user wants platform-appropriate FFmpeg installation guidance.
- After the user installs FFmpeg, rerun the dependency checks before continuing.

## What this Skill Can Do

| Capability | Resource | Description |
|---|---|---|
| Image generation | `pony_flash.images` | Text-to-image, image editing with mask/reference images |
| Video generation | `pony_flash.video` | Text-to-video, first-frame-to-video, OmniHuman, Motion Transfer |
| Speech synthesis | `pony_flash.speech` | Text-to-speech with voice cloning, emotion control, speed, pitch |
| Music generation | `pony_flash.music` | Text-to-music with lyrics, style, instrumental mode, continuation |
| Model listing | `pony_flash.models` | List available models, get model details and supported modes |
| File management | `pony_flash.files` | Upload, list, get, delete files |
| Account | `pony_flash.account` | Check credit balance, get recharge link |
| Local media editing | `scripts/media_ops.sh` | Clip, concat, transcode, extract audio, frame capture |
| FFmpeg environment checks | `scripts/check_ffmpeg.sh` | Detect ffmpeg / ffprobe and subtitle capabilities |
| Subtitle font prep | `scripts/ensure_subtitle_fonts.sh` | Keep a reusable local copy of the default subtitle font when explicitly requested |
| ASS subtitle prep | `scripts/build_ass_subtitles.py` | Adaptive ASS subtitle generation with pre-wrapping |

## Creative Playbooks (production workflows)

The `playbooks/` directory contains **Creative Playbooks** — step-by-step production workflow guides for specific content types. Playbooks act as a director layer: they tell you **what to create and in what order**, while this SKILL.md tells you **how to execute generation and editing**.

### When to use a playbook

1. **User explicitly requests a playbook by name** → Read the corresponding file from `playbooks/` and follow its workflow.
2. **User asks to see available playbooks** → Read [playbooks/INDEX.md](playbooks/INDEX.md) and display the full list.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ponyflash/ponyflash-skill](https://github.com/ponyflash/ponyflash-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
