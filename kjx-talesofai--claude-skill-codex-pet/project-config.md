---
trigger: always_on
description: Create, repair, validate, preview, and package Codex-compatible animated pet spritesheets from character art, screenshots, generated images, or visual references. Use when a user wants to hatch a Codex pet, create a custom animated pet, or build a built-in pet asset with an 8x9 atlas, transparent unused cells, row-by-row animation prompts, QA contact sheets, preview videos, and pet.json packaging. Requires an image generator with text-prompt + reference-image grounding capabilities. Uses bundled
---


# Hatch Pet

## Overview

Create a Codex-compatible animated pet from a concept, one or more reference images, or both. This skill owns pet-specific prompt planning, animation rows, frame extraction, atlas geometry, QA, previews, and packaging. It delegates visual generation to whatever image generation tool is available in the current environment.

User-facing inputs are optional. If the user omits a pet name, infer one from the concept or reference filenames; if that is not possible, choose a short appropriate name. If the user omits a description, infer one from the concept or references. If the user omits reference images, generate the base pet from text first, then use that base as the canonical reference for every animation row.

## Required Image Generation Capabilities

The image generator used with this skill must support:

1. **Text-to-image generation** from detailed prompts.
2. **Reference image grounding** (attaching input images to guide output). This is required for row-strip generation — every row after the base must use grounding images to preserve identity.
3. **PNG output** (or convertible to PNG). The deterministic pipeline ingests PNGs.
4. **Reasonable prompt adherence** for precise layouts: the generator should be able to produce a horizontal strip with an exact frame count, flat green background, and evenly spaced poses when instructed.

If the available image generator does not support reference image grounding, it can only be used for the base job (which may be prompt-only). All row-strip jobs require grounding.

**Tested providers:** This skill has been tested with GPT Image 2 via proxy (strong reference grounding, URL output) and Google Gemini image generation (text-only for base, no reference support for rows). Any image generator meeting the four capabilities above should work.

## Generation Delegation

Use the environment's available image generation tool for all visual generation.

When invoking an image generator from this skill, pass the generated pet prompt as the authoritative visual spec. Do not wrap it in the generic shared prompt schema and do not add extra polish, hero-art, photo, product, or illustration-style augmentation. Pet prompts should stay terse, sprite-specific, and digital-pet oriented; only add role labels for input images and any essential user constraint.

Use this skill's scripts for deterministic work only: preparing prompts and manifests, ingesting selected image generator outputs, extracting frames, validating rows, composing the final atlas, creating QA media, and packaging.

Hard boundary: do not create, draw, tile, warp, mirror, or synthesize pet visuals with local Python/Pillow scripts, SVG, canvas, HTML/CSS, or other code-native art as a substitute for the image generator. For a normal pet run, expect up to 10 visual generation jobs: 1 base pet plus 9 row-strip jobs. The only exception is `running-left`, which may be derived by mirroring `running-right` only after `running-right` has been generated, visually inspected, and explicitly approved as safe to mirror. If mirroring is not appropriate, generate `running-left` as a normal grounded row. If those calls are too expensive, blocked, or unavailable, stop and explain the blocker instead of fabricating row strips locally.

Do not mark visual jobs complete by editing `imagegen-jobs.json`, copying files into `decoded/`, or writing helper scripts that populate row outputs. Use `record_imagegen_result.py` for selected image generator outputs. The deterministic scripts may only process already-generated visual outputs.

Only the base job may be prompt-only. Every row-strip job must use the input images listed in `imagegen-jobs.json`, including the canonical base reference created after the base job is recorded. Treat any row generation without attached grounding images as invalid.

## Image Generation Integration Notes

### Discovering the local image generator

Before generating, check what image generation tools are available in the current environment. Common patterns:

- Look for a skill or command in the agent's skill directory (e.g., `~/.claude/skills/`, `.claude/skills/`)
- Check for CLI tools like `nb-generate`, `gpt-image`, or custom wrappers
- Check environment variables like `OPENAI_API_KEY`, `GPT_IMAGE_API_KEY`

### Handling URL outputs

If the image generator returns a URL instead of writing a local file:

1. Download the image from the URL.
2. Save it as a local PNG.
3. Pass the local PNG path to `record_imagegen_result.py`.

### Enforcing frame consistency in prompts

Frame size and identity drift between frames is the most common quality issue. Every row prompt should include **explicit consistency constraints** tailored to the animation state:

**For all rows:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kjx-talesofai/claude-skill-codex-pet](https://github.com/kjx-talesofai/claude-skill-codex-pet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
