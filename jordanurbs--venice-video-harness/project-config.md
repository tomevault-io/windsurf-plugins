---
trigger: always_on
description: This workspace is an agent-first, Venice-optimized harness for **consistency-first video creation at any length and format**.
---

# Venice Video Harness

This workspace is an agent-first, Venice-optimized harness for **consistency-first video creation at any length and format**.

It is meant to be operated through natural language in an IDE like Cursor or VS Code with an agent such as Claude Code. The user should not be asked to run terminal commands manually. The agent reads the rules, selects the right playbooks, and executes code as needed.

## What This Harness Does

1. Helps an agent plan and execute consistency-first Venice video workflows
2. Supports recurring characters, locked visual systems, and reference-driven generation
3. Provides reusable orchestration through `CLAUDE.md`, `.claude/commands/`, `.claude/agents/`, and `.claude/skills/`
4. Includes a comprehensive model registry covering 50+ Venice video, image, audio, and music models
5. Includes a working narrative reference implementation in `src/mini-drama/`
6. Preserves generated media by archiving instead of destructively replacing

## Supported Use Cases

This harness is not limited to any single video format. It supports:

- **Episodic series** (drama, comedy, documentary, educational)
- **Trailers and teasers**
- **Branded cinematic sequences**
- **Product launch videos**
- **Recurring-character social content**
- **Narrative explainers**
- **Style-locked creative campaigns**
- **Long-form content** (assemble multi-shot sequences of any length)
- **Any Venice workflow where visual continuity matters**

## How To Operate

The intended interface is:
- Natural-language requests to the agent
- Orchestration rules in `CLAUDE.md`
- Workflow playbooks in `.claude/commands/`
- Reusable Venice knowledge in `.claude/skills/`
- Underlying TypeScript and script execution in `src/` and `scripts/`

The CLI and scripts are the execution layer underneath the harness, not the primary user interface.

## Venice API Coverage

### Video Endpoints

| Endpoint | Purpose | Module |
|----------|---------|--------|
| `POST /video/queue` | Queue video generation | `src/venice/video.ts` |
| `POST /video/retrieve` | Poll/download result | `src/venice/video.ts` |
| `POST /video/quote` | Get cost estimate | `src/venice/video.ts` |
| `POST /video/complete` | Cleanup after download | `src/venice/video.ts` |

### Image Endpoints

| Endpoint | Purpose | Module |
|----------|---------|--------|
| `POST /image/generate` | Text-to-image | `src/venice/generate.ts` |
| `POST /image/multi-edit` | Layered multi-image editing | `src/venice/multi-edit.ts` |
| `POST /image/upscale` | AI upscaling | `src/venice/edit.ts` |
| `POST /image/background-remove` | Background removal | `src/venice/edit.ts` |
| `POST /images/edit` | **DEPRECATED** (May 2025) | `src/venice/edit.ts` |

### Audio Endpoints

| Endpoint | Purpose | Module |
|----------|---------|--------|
| `POST /audio/speech` | Text-to-speech (Kokoro, Qwen3) | `src/venice/audio.ts` |
| `POST /audio/queue` | Queue music/SFX generation | `src/venice/audio.ts` |
| `POST /audio/retrieve` | Poll/download audio result | `src/venice/audio.ts` |
| `POST /audio/complete` | Cleanup after download | `src/venice/audio.ts` |

### Chat Endpoint

| Endpoint | Purpose | Module |
|----------|---------|--------|
| `POST /chat/completions` | Vision-based QA, script generation | `src/venice/client.ts` |

## Model Registry

The full model registry lives in `src/venice/models.ts` with typed specs for every model. Key categories:

### Video Models (50+ models)

**Action / Movement / Dialogue:**
- `kling-v3-pro-image-to-video` (3-15s, audio, `end_image_url`)
- `kling-o3-pro-image-to-video` (3-15s, audio, `end_image_url`)
- `kling-2.6-pro-image-to-video` (5-10s, audio, `end_image_url`)
- `wan-2.6-image-to-video` (5-15s, 1080p, audio, `audio_url` input)
- `sora-2-pro-image-to-video` (4-12s, 1080p, audio)

**Atmosphere / Establishing / Mood:**
- `seedance-2-0-image-to-video` (default atmosphere + action model, 4-15s, 720p, native stereo audio, #1 ranked)
- `veo3.1-fast-image-to-video` (4-8s, up to 4K, audio)
- `veo3-fast-image-to-video` (8s, audio)
- `pixverse-v5.6-image-to-video` (5-8s, up to 1080p, audio)

**Character Consistency (Reference-to-Video):**
- `seedance-2-0-reference-to-video` (default R2V, 4-15s, `reference_image_urls` up to 4, `@Image` tags, native audio, #1 ranked)
- `kling-o3-standard-reference-to-video` (fallback for 3+ characters, 3-15s, `elements`, `reference_image_urls`, `scene_image_urls`)
- `kling-o3-pro-reference-to-video` (3-15s, full reference support)

**Long Duration:**
- `longcat-image-to-video` / `longcat-distilled-image-to-video` (up to **30s**, no audio)
- `ltx-2-fast-image-to-video` / `ltx-2-v2-3-fast-image-to-video` (up to **20s**, up to 4K)
- `ltx-2-19b-full-image-to-video` (up to **18s**, audio)

**Budget / Fast:**
- `wan-2.6-flash-image-to-video` (5-15s, fast)
- `kling-v3-standard-image-to-video` (3-15s)
- `grok-imagine-image-to-video` (5-15s)

### Video Model Capabilities

| Capability | Models |
|-----------|--------|
| `elements` (structured @Element refs) | Kling O3 R2V (standard + pro) |
| `reference_image_urls` (flat ref array) | Seedance 2.0 R2V, Kling O3 R2V, Vidu Q3 |
| `scene_image_urls` (environment refs) | Kling O3 R2V (standard + pro) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jordanurbs/venice-video-harness](https://github.com/jordanurbs/venice-video-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
