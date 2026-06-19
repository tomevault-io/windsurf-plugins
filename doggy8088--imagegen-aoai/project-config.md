---
trigger: always_on
description: Generate or edit raster images when the task benefits from AI-created bitmap visuals such as photos, illustrations, textures, sprites, mockups, or transparent-background cutouts. Use when Codex should create a brand-new image, transform an existing image, or derive visual variants from references, and the output should be a bitmap asset rather than repo-native code or vector. Do not use when the task is better handled by editing existing SVG/vector/code-native assets, extending an established ic
---


# Image Generation Skill

Generates or edits images for the current project (for example website assets, game assets, UI mockups, product mockups, wireframes, logo design, photorealistic images, or infographics).

## Execution mode and rules

This skill uses the bundled Azure OpenAI CLI at `scripts/image_gen.js`. GitHub Copilot CLI does not provide built-in image-generation or image-viewing tools, so do not reference or attempt to use nonexistent built-ins.

The CLI exposes three subcommands:

- `generate`
- `edit`
- `generate-batch`

Rules:
- Use `scripts/image_gen.js` for normal image generation and editing requests.
- If the user explicitly asks for a transparent image/background, start with the default `gpt-image-2` chroma-key workflow: prompt for a flat removable chroma-key background, then remove it locally with the installed helper at `$CODEX_HOME/skills/.system/imagegen/scripts/remove_chroma_key.js`.
- Never silently switch from CLI `gpt-image-2` to CLI `gpt-image-1.5`. Treat this as a model/path downgrade and ask the user before doing it, unless the user has already explicitly requested `gpt-image-1.5` or `scripts/image_gen.js`.
- If a transparent request appears too complex for clean chroma-key removal, asks for true/native transparency, or local removal fails validation, explain that true transparency requires CLI `gpt-image-1.5 --background transparent --output-format png` because `gpt-image-2` does not support `background=transparent`, then ask whether to proceed. Run the `gpt-image-1.5` path only after the user confirms.
- The word `batch` by itself does not require `generate-batch`. Use `generate-batch` when a multi-asset request is naturally represented as a JSONL batch; otherwise run individual CLI commands.
- Live API calls require Azure OpenAI credentials and network access.
- Use the bundled `scripts/image_gen.js` workflow. Do not create one-off SDK runners.
- Never modify `scripts/image_gen.js`. If something is missing, ask the user before doing anything else.

Output policy:
- Write generated assets directly to the requested path, or to `output/imagegen/` when the user does not specify a path.
- If the image is meant for the current project, save the final selected image in the workspace before finishing.
- Do not overwrite an existing asset unless the user explicitly asked for replacement; otherwise create a sibling versioned filename such as `hero-v2.png` or `item-icon-edited.png`.

Shared prompt guidance lives in `references/prompting.md` and `references/sample-prompts.md`.

CLI docs/resources:
- `references/cli.md`
- `references/image-api.md`
- `references/codex-network.md`
- `scripts/image_gen.js`

Local post-processing helper:
- `$CODEX_HOME/skills/.system/imagegen/scripts/remove_chroma_key.js`: removes a flat chroma-key background from a generated image and writes a PNG/WebP with alpha. Prefer auto-key sampling, soft matte, and despill for antialiased edges.

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
- If the user provides images only as references for style, composition, mood, or subject guidance, treat the request as **generate**.
- If the user provides no images, treat the request as **generate**.

Edit semantics:
- CLI edit mode accepts local image file paths via `--image`.
- If the user provides attached images or references from the conversation, save them locally first if the environment supports that; otherwise ask the user for local file paths.
- Use repeated `--image` flags for multi-image edits. Their order is meaningful, so label each image by index and role in the prompt.
- For edits, preserve invariants aggressively and save non-destructively by default.

Execution strategy:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [doggy8088/imagegen-aoai](https://github.com/doggy8088/imagegen-aoai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
