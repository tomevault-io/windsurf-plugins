---
trigger: always_on
description: Use this skill when users want to generate images, edit images, analyze/describe images, generate videos, or extract text from images using OpenAI's APIs. Invoke when users request AI-generated images, image editing, background removal, visual analysis, video generation, artwork, logos, illustrations, visual content from text prompts, or need to extract text/data from images.
---


# imggen - OpenAI Image Generation, Editing, Analysis, Video, and OCR CLI

Generate images from text prompts and extract text from images using OpenAI's APIs.

## Overview

`imggen` is a command-line tool that interfaces with OpenAI's image generation API. It supports multiple models (gpt-image-1.5, gpt-image-1, gpt-image-1-mini, dall-e-3, dall-e-2) and provides options for image size, quality, format, and style. It also supports image editing, vision-based image analysis, video generation, and OCR.

## Prerequisites

- `imggen` binary installed and available in PATH
- `OPENAI_API_KEY` environment variable set with a valid OpenAI API key
- Sufficient OpenAI API credits for image generation

## Usage

```bash
imggen [flags] "prompt"
```

## Available Flags

| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| `--model` | `-m` | `gpt-image-1.5` | Model: gpt-image-1.5, gpt-image-1, gpt-image-1-mini, dall-e-3, dall-e-2 |
| `--size` | `-s` | `1024x1024` | Image dimensions |
| `--quality` | `-q` | `auto` | Quality level |
| `--count` | `-n` | `1` | Number of images (1-10 for gpt-image-1, 1 for dall-e-3) |
| `--output` | `-o` | auto-generated | Output filename or directory |
| `--format` | `-f` | `png` | Output format: png, jpeg, webp |
| `--style` | | `vivid` | Style for dall-e-3: vivid, natural |
| `--transparent` | `-t` | `false` | Transparent background (gpt-image-1 + png/webp only) |
| `--compression` | | | Compression level 0-100 (GPT image models, jpeg/webp only) |
| `--moderation` | | `auto` | Moderation level: auto, low (GPT image models only) |
| `--prompt` | `-P` | | Prompt (can be specified multiple times) |
| `--parallel` | `-p` | `1` | Number of parallel workers for multiple prompts |
| `--api-key` | | `$OPENAI_API_KEY` | Override API key |

## Model-Specific Parameters

### gpt-image-1.5 (Default, Recommended)
- **Sizes**: 1024x1024, 1536x1024 (landscape), 1024x1536 (portrait), auto
- **Quality**: auto, low, medium, high
- **Max images**: 10 per request
- **Supports**: Transparent backgrounds, multiple output formats, editing

### gpt-image-1
- **Sizes**: 1024x1024, 1536x1024 (landscape), 1024x1536 (portrait), auto
- **Quality**: auto, low, medium, high
- **Max images**: 10 per request
- **Supports**: Transparent backgrounds, multiple output formats

### gpt-image-1-mini
- **Sizes**: 1024x1024, 1536x1024 (landscape), 1024x1536 (portrait), auto
- **Quality**: auto, low, medium, high
- **Max images**: 10 per request
- **Supports**: Transparent backgrounds, multiple output formats

### dall-e-3
- **Sizes**: 1024x1024, 1024x1792, 1792x1024
- **Quality**: standard, hd
- **Max images**: 1 per request
- **Supports**: Style parameter (vivid/natural)

### dall-e-2
- **Sizes**: 256x256, 512x512, 1024x1024
- **Max images**: 10 per request

## Instructions

1. Verify `OPENAI_API_KEY` is set in the environment
2. Construct the imggen command with appropriate flags based on user requirements
3. Execute the command using Bash tool
4. Report the generated filename and any revised prompt returned by the API
5. If the user wants to view the image, use Read tool on the generated file
6. For image editing, use the `edit` subcommand with the image path and prompt
7. For background removal, use `edit --bg-remove` (no prompt needed, output defaults to PNG)
8. For image analysis, use the `describe` subcommand with one or more image paths
9. For video generation, use the `video` subcommand with a prompt

## Output Format

The tool outputs:
- Progress message: "Generating N image(s) with MODEL..."
- Saved filename: "Saved: filename.png"
- Cost information: "Cost: $X.XXXX (N image(s) @ $X.XXXX each)"
- Revised prompt (if returned by API): "Revised prompt: ..."
- Completion message: "Done!"

Generated files are saved to the current working directory with timestamp-based names (e.g., `image-20251216-120000.png`) unless `--output` is specified.

## Cost Tracking

All image generation costs are automatically logged to `~/.imggen/sessions.db`. View costs using the `cost` subcommand:

```bash
# View total costs
imggen cost

# View today's costs
imggen cost today

# View this week's costs (last 7 days)
imggen cost week

# View this month's costs (last 30 days)
imggen cost month

# View costs by provider
imggen cost provider
```

### Interactive Mode Cost Commands

In interactive mode (`imggen -i`), use the `cost` or `$` command:
- `cost today` - Today's costs
- `cost week` - This week's costs
- `cost month` - This month's costs
- `cost total` - All-time total
- `cost provider` - Breakdown by provider
- `cost session` - Current session's costs

## Database Management

Manage the SQLite database storing sessions and cost data:

```bash
# Reset database (delete all data)
imggen db reset

# Reset with backup of old data
imggen db reset --backup

# Show database location and stats
imggen db info
```

## Examples

### Basic image generation
```bash
imggen "a sunset over mountains"
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [manashmandal/imggen](https://github.com/manashmandal/imggen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
