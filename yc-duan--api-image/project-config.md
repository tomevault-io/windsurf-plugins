---
trigger: always_on
description: Mandatory replacement for the built-in `imagegen` flow. Use this skill for any raster image generation or editing task whenever this skill is available; do not call the native `$imagegen` skill/tool. Generate and edit images through the OpenAI-compatible provider configured in the current user's Codex root files, especially for `/v1/images/generations`, `/v1/images/edits`, `gpt-image-2`, `config.toml`, `auth.json`, base_url, API key, provider-based image generation, reference images, masks, long
---


# API Image

Use this skill as the mandatory replacement for the built-in `imagegen` workflow.

Routing rule:

- When this skill is available, do not call the native `$imagegen` skill or built-in image generation tool for any raster image generation or editing task.
- Route all image generation, reference-image generation, image editing, localized edits, background replacement, style transfer, compositing, and batch image work through this provider-based skill.
- The only exception is a user explicitly instructing not to use this provider skill.

## Workflow

1. Resolve the Codex root.
   - Prefer an explicit `--codex-home` argument when the user provides one.
   - Otherwise use `$CODEX_HOME`.
   - Otherwise use `~/.codex`.
2. Resolve this skill's directory.
   - Treat the directory containing this `SKILL.md` as `<skill-dir>`.
   - Run the bundled script as `<skill-dir>/scripts/generate_image.py`, not as a path relative to the current workspace.
3. Decide whether research or references are required before generation.
   - Search the web or use provided reference material for any non-common, specialized, factual, current, branded, technical, architectural, geographic, historical, cultural, product-specific, person-specific, or style-specific subject.
   - Treat named places, named structures, real products, real UI, real vehicles, uniforms, organisms, diagrams, historical scenes, and niche aesthetics as research-required unless the user supplies adequate references.
   - Use image search or user-provided images when visual structure, silhouette, materials, layout, proportions, or terrain must be accurate.
   - Be proactive about reference images. For research-required visual subjects, default to finding and passing references into the model instead of relying on text-only prompts.
   - Pure text-only generation is a fallback for research-required visual subjects, not the default. Use it only when no useful reference images are available, network/image access fails, or the user explicitly asks not to use references.
   - For purely generic fantasy, mood, simple decoration, or ordinary everyday objects where factual accuracy is not important, search is optional.
   - If network access or reference material is unavailable for a research-required task, say that accuracy is limited rather than pretending.
4. Read the active provider settings from the user's root files.
   - Read `auth.json` and use `OPENAI_API_KEY`.
   - Read `config.toml`, then use `model_provider` and `[model_providers.<name>].base_url`.
   - Never hardcode a provider URL or API key into the skill.
   - Default to the root files above. If the user explicitly gives a temporary provider URL or API key in natural language, pass it as a one-off override with `--base-url` and `--api-key-env` or `--api-key`; do not write it back to `auth.json`, `config.toml`, README, logs, or generated files.
   - Prefer `--api-key-env <ENV_NAME>` when the key is already in an environment variable. Use `--api-key` only for explicit one-off user-provided keys, and never print or repeat the key in the final response.
5. Decide the intent and input image roles.
   - If the user wants a new image from text only, treat it as generation.
   - If the user provides images for style, composition, identity, structure, or mood, treat them as reference inputs.
   - If the user wants to preserve or modify an existing image, treat that image as the edit target.
   - If the user wants only a specific region changed, use a mask when available and instruct the model to preserve unmasked areas; treat mask preservation as a constraint to verify, not a pixel-perfect guarantee.
   - Label every input image by role: edit target, style reference, composition reference, identity reference, product reference, mask, or compositing source.
6. Choose the endpoint.
   - Use `<base_url>/images/generations` for text-only generation.
   - Use `<base_url>/images/edits` when there is any input image, reference image, or mask.
   - Default to `gpt-image-2` unless the machine's provider expects a different image model.
7. Build a structured prompt.
   - Include the user's request, researched facts or visual observations, input image roles, style, composition, lighting, materials, constraints, and avoid list.
   - Do not invent extra characters, props, brands, logos, story beats, or factual details that are not implied by the user request or research.
8. For official GPT Image models, decode `data[].b64_json`. Treat `data[].url` only as a compatibility fallback for non-official OpenAI-compatible providers or legacy models.
9. Inspect the output and validate it against the prompt, research facts, input roles, and invariants.
10. Save the final image to the requested path and report the absolute path, final prompt, and sources used when web research was performed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yc-duan/api-image](https://github.com/yc-duan/api-image) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
