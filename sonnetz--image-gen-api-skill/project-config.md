---
trigger: always_on
description: Generate or edit raster images through an OpenAI-compatible Images API, model relay, or CC Switch-managed Codex provider. Use by default instead of the built-in imagegen skill when Codex is running with a custom/non-OpenAI provider, a non-official base_url, IMAGE_API_BASE_URL/OPENAI_BASE_URL, or when the user asks to use CC Switch, a gateway, relay, API, or non-OpenAI image model. Do not use when the task is better handled by SVG/vector/code-native assets.
---


# Image Gen API Skill

Generates or edits images for the current project through `scripts/image_gen.py`, using an OpenAI-compatible Images API endpoint resolved from environment variables or Codex/CC Switch configuration.

## Default routing

Use this skill as the default image-generation path when any of these are true:

- Codex `~/.codex/config.toml` has `model_provider` set to a custom provider or a provider whose `base_url` is not the official OpenAI API URL.
- The environment contains `IMAGE_API_BASE_URL`, `IMAGE_GEN_API_BASE_URL`, or `OPENAI_BASE_URL`.
- The user asks for CC Switch, a model gateway/relay, API-based image generation, custom `base_url`, custom model, or non-OpenAI image provider.

If Codex is using the built-in official provider and the user did not ask for API/gateway behavior, defer to the system `$imagegen` skill.

## Execution model

This skill has one primary execution mode:

- **API CLI mode:** use `scripts/image_gen.py` for generation, editing, and batch generation. It resolves `base_url`, API key, and model from explicit CLI args, environment variables, or Codex config.

The CLI exposes three subcommands:

- `generate`
- `edit`
- `generate-batch`

Rules:
- Use `scripts/image_gen.py` directly; do not create one-off SDK runners.
- Do not hard-code provider URLs, model names, or secrets in the skill.
- Never print API keys or bearer tokens. Dry-runs may show `base_url`, model, and config source only.
- If the provider does not support `/v1/images/generations` or `/v1/images/edits`, report that capability mismatch and ask the user to switch to an image-capable provider/model.
- For many distinct assets, use `generate-batch`; do not use `n` as a substitute for separate prompts. `n` is for variants of one prompt.

## Config resolution

`scripts/image_gen.py` resolves configuration in this order:

1. Explicit args: `--base-url`, `--model`, `--api-key-env`, `--provider`, `--codex-config`.
2. Image-specific environment variables: `IMAGE_API_BASE_URL`, `IMAGE_API_KEY`, `IMAGE_API_MODEL`, plus `IMAGE_GEN_API_*` aliases.
3. OpenAI-compatible environment variables: `OPENAI_BASE_URL`, `OPENAI_API_KEY`, `OPENAI_IMAGE_MODEL`.
4. Codex config: `${CODEX_HOME:-~/.codex}/config.toml`, including CC Switch-managed `[model_providers.<name>]` values such as `base_url` and `experimental_bearer_token`.

If a Codex provider `base_url` is a bare host such as `https://gateway.example.com`, the CLI normalizes it to `https://gateway.example.com/v1` for image API calls. Pass `--base-url` to override this exactly.

The image model comes from `--model`, `IMAGE_API_MODEL`/`IMAGE_GEN_API_MODEL`/`OPENAI_IMAGE_MODEL`, or the built-in default `gpt-image-2`. The CLI does not use the current Codex chat model as the default image model.

Save-path policy:
- Default one-off output path: `output/image-gen-api/output.png`.
- If the user names a destination, use that destination.
- If the image is meant for the current project, save the final asset in the workspace and update consuming code or references.
- Do not overwrite an existing asset unless the user explicitly asked for replacement; otherwise create a sibling versioned filename such as `hero-v2.png` or `item-icon-edited.png`.

Shared prompt guidance lives in `references/prompting.md` and `references/sample-prompts.md`.

API docs/resources:
- `references/cli.md`
- `references/image-api.md`
- `references/codex-network.md`
- `scripts/image_gen.py`

Local post-processing helper:
- `$CODEX_HOME/skills/.system/imagegen/scripts/remove_chroma_key.py`: removes a flat chroma-key background from a generated image and writes a PNG/WebP with alpha. Prefer auto-key sampling, soft matte, and despill for antialiased edges.

## When to use
- Generate a new image (concept art, product shot, cover, website hero)
- Generate a new image using one or more reference images for style, composition, or mood
- Edit an existing image (inpainting, lighting or weather transformations, background replacement, object removal, compositing, transparent background)
- Produce many assets or variants for one task

## When not to use
- Extending or matching an existing SVG/vector icon set, logo system, or illustration library inside the repo
- Creating simple shapes, diagrams, wireframes, or icons that are better produced directly in SVG, HTML/CSS, or canvas
- Making a small project-local asset edit when the source file already exists in an editable native format
- Any task where the user clearly wants deterministic code-native output instead of a generated bitmap

## Decision tree

Think about two separate questions:

1. **Intent:** is this a new image or an edit of an existing image?
2. **Execution strategy:** is this one asset or many assets/variants?

Intent:
- If the user wants to modify an existing image while preserving parts of it, treat the request as **edit**.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SonnetZ/image-gen-api-skill](https://github.com/SonnetZ/image-gen-api-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
