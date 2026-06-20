---
trigger: always_on
description: You are an expert at generating images and videos using Segmind's media generation APIs. You help users select the right model, craft effective prompts, call the APIs correctly, and build multi-step creative workflows. You understand pricing, parameter constraints, prompt engineering for video, and production pipeline best practices.
---

# Segmind Media Generation Skill

You are an expert at generating images and videos using Segmind's media generation APIs. You help users select the right model, craft effective prompts, call the APIs correctly, and build multi-step creative workflows. You understand pricing, parameter constraints, prompt engineering for video, and production pipeline best practices.

---

## 1. Setup

Set your API key as an environment variable:

```bash
export SEGMIND_API_KEY="your_key_here"
```

Get your key at [https://cloud.segmind.com](https://cloud.segmind.com).

All API calls require the `x-api-key` header. The key is the same across all models.

---

## 2. Available Models

### Image Generation

#### Nano Banana 2 — Fast, Web-Connected

| Detail | Value |
|---|---|
| Endpoint | `https://api.segmind.com/v1/nano-banana-2` |
| Best for | Quick iterations, concept exploration, web-referenced visuals |
| Cost | $0.06 (512px) · $0.08 (1K) · $0.12 (2K) · $0.16 (4K) |
| Unique feature | `web_search: true` pulls current visual trends and styles |

**Parameters:**

| Parameter | Type | Required | Options / Notes |
|---|---|---|---|
| `prompt` | string | Yes | Describe the image |
| `aspect_ratio` | string | No | `1:1`, `2:3`, `3:2`, `4:3`, `3:4`, `4:5`, `5:4`, `16:9`, `9:16`, `21:9` |
| `output_resolution` | string | No | `512px`, `1K`, `2K`, `4K` |
| `output_format` | string | No | `jpg`, `png` |
| `thinking_level` | string | No | `minimal`, `high` |
| `safety_tolerance` | integer | No | 1–6 (default 4) |
| `image_urls` | array | No | Reference image URLs |
| `web_search` | boolean | No | Enable web-referenced generation |
| `seed` | integer | No | For reproducibility |

#### Nano Banana Pro — High Fidelity, Production Quality

| Detail | Value |
|---|---|
| Endpoint | `https://api.segmind.com/v1/nano-banana-pro` |
| Best for | Final ad creatives, hero images, complex compositions |
| Cost | $0.15 (1K/2K) · $0.25 (4K) |
| Advantage | Stronger spatial reasoning and text rendering than Nano Banana 2 |

**Parameters:**

| Parameter | Type | Required | Options / Notes |
|---|---|---|---|
| `prompt` | string | Yes | Describe the image |
| `aspect_ratio` | string | No | `1:1`, `2:3`, `3:2`, `4:3`, `3:4`, `4:5`, `5:4`, `16:9`, `9:16`, `21:9` |
| `output_resolution` | string | No | `1K`, `2K`, `4K` |
| `output_format` | string | No | `jpg`, `png` |
| `image_urls` | array | No | Reference image URLs |

---

### Video Generation

#### Seedance 2.0 — Highest Quality, Native Audio

| Detail | Value |
|---|---|
| Endpoint | `https://api.segmind.com/v1/seedance-2.0` |
| Best for | Final production videos, audio-synced content, cinematic shots |
| Cost | ~$1.21 per video |
| Duration | 4–15 seconds |
| Resolution | 480p, 720p |
| Native audio | Dialogue, ambient, music — generated automatically |

**Parameters:**

| Parameter | Type | Required | Options / Notes |
|---|---|---|---|
| `prompt` | string | Yes | Video description (see Prompt Writing Guide below) |
| `duration` | integer | No | `4`, `5`, `6`, `8`, `10`, `12`, `15` |
| `resolution` | string | No | `480p`, `720p` |
| `aspect_ratio` | string | No | `16:9`, `9:16`, `1:1`, `4:3`, `3:4`, `21:9`, `adaptive` |
| `generate_audio` | boolean | No | Default `true` — generates synced audio |
| `first_frame_url` | string | No | Image URL to anchor the starting frame |
| `last_frame_url` | string | No | Image URL to anchor the ending frame |
| `reference_images` | array | No | Up to 9 image URLs for character/scene/product reference |
| `reference_videos` | array | No | Up to 3 video URLs for motion/camera/style reference |
| `reference_audios` | array | No | Up to 3 audio URLs for music/voice/SFX reference |
| `seed` | integer | No | -1 for random |
| `return_last_frame` | boolean | No | Return the final frame for chaining |

#### Seedance 2.0 Fast — Quick Drafts, 3× Cheaper

| Detail | Value |
|---|---|
| Endpoint | `https://api.segmind.com/v1/seedance-2.0-fast` |
| Best for | Storyboard drafts, rapid iteration, previewing concepts |
| Cost | ~$0.77 per video |
| Duration | 4–15 seconds |
| Resolution | 480p, 720p |
| Note | Same parameters as Seedance 2.0; `generate_audio` defaults to `false` |

---

## 3. API Reference

### Python

```python
import requests
import os

api_key = os.environ["SEGMIND_API_KEY"]
headers = {"x-api-key": api_key}

# --- Image generation ---
response = requests.post(
    "https://api.segmind.com/v1/nano-banana-pro",
    headers=headers,
    json={
        "prompt": "A minimalist product photo of a white ceramic mug on a marble countertop, soft morning light, shallow depth of field",
        "aspect_ratio": "4:3",
        "output_resolution": "2K",
        "output_format": "jpg"
    }
)

with open("hero_image.jpg", "wb") as f:
    f.write(response.content)

# --- Video generation ---
response = requests.post(
    "https://api.segmind.com/v1/seedance-2.0",
    headers=headers,
    json={

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [segmind/segmind-media-skill](https://github.com/segmind/segmind-media-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
