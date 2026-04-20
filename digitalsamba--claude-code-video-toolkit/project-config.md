---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with this video production toolkit.
---

# claude-code-video-toolkit

This file provides guidance to Claude Code (claude.ai/code) when working with this video production toolkit.

## Overview

**claude-code-video-toolkit** is an AI-native video production workspace. It provides Claude Code with the skills, commands, and tools to create professional videos from concept to final render.

**Key capabilities:**
- Programmatic video creation with Remotion (React-based)
- AI voiceover generation with ElevenLabs or Qwen3-TTS
- AI music generation with ACE-Step 1.5 (text-to-music, vocals, covers, stems)
- Browser demo recording with Playwright
- Asset processing with FFmpeg

## Directory Structure

```
claude-code-video-toolkit/
├── .claude/
│   ├── skills/          # Domain knowledge for Claude
│   └── commands/        # Guided workflows
├── tools/               # Python CLI automation
├── templates/           # Video templates
│   ├── sprint-review/   # Sprint review video template
│   └── product-demo/    # Marketing/product demo template
├── brands/              # Brand profiles (colors, fonts, voice)
├── projects/            # Your video projects go here (gitignored)
├── examples/            # Curated showcase projects (shared)
├── assets/              # Shared assets (voices, images)
├── playwright/          # Browser recording infrastructure
├── docs/                # Documentation
└── _internal/           # Toolkit metadata & registry
```

## Registry

`_internal/toolkit-registry.json` is the canonical source for all skills, commands, tools, templates, components, transitions, and cloud endpoints — including their paths, status, options, presets, and env vars. Consult it for structured data. This file focuses on **workflow guidance, patterns, and knowledge** that the registry can't capture.

## Quick Start

**First-time setup (optional, ~5 minutes):**
```
/setup
```

Walks through cloud GPU, file transfer (R2), and voice configuration. Most features are free. Skip this if you just want to render videos with Node.js.

**Work on a video project:**
```
/video
```

This command will:
1. Scan for existing projects (resume or create new)
2. Choose template (sprint-review, product-demo)
3. Choose brand (or create one with `/brand`)
4. Plan scenes interactively
5. Create project with VOICEOVER-SCRIPT.md

**Multi-session support:** Projects span multiple sessions. Run `/video` to resume where you left off. Each project tracks its phase, scenes, assets, and session history in `project.json`.

**Or manually:**
```bash
cp -r templates/sprint-review projects/my-video
cd projects/my-video
npm install
npm run studio   # Preview
npm run render   # Export
```

> **Note:** After creating or modifying commands/skills, restart Claude Code to load changes.

## Templates

Templates live in `templates/`. Each is a standalone Remotion project. See registry `templates` section for the full list.

### sprint-review
Config-driven sprint review videos with theme system, config-driven content (`sprint-config.ts`), pre-built slides (Title, Overview, Summary, Credits), demo components (single video, split-screen), and audio integration.

### product-demo
Marketing/product demo videos with dark tech aesthetic, scene-based composition (title, problem, solution, demo, stats, CTA), animated background, Narrator PiP, browser/terminal chrome, and stats cards with spring animations.

## Brand Profiles

Brands live in `brands/`. Each defines visual identity:

```
brands/my-brand/
├── brand.json    # Colors, fonts, typography
├── voice.json    # ElevenLabs voice settings
└── assets/       # Logo, backgrounds
```

See `docs/creating-brands.md` for details.

## Shared Components

Reusable video components in `lib/components/`. See registry `components` section for the full list with descriptions. Import in templates via:

```tsx
import { AnimatedBackground, SlideTransition, Label } from '../../../../lib/components';
```

## Python Tools

Audio, video, and image tools in `tools/`. See registry `tools` section for the full catalog with descriptions, options, presets, and env vars. Every tool supports `--help`.

```bash
# Setup
pip install -r tools/requirements.txt
```

**Important: always invoke tools from the toolkit root directory.** When working inside a project (`projects/my-video/`), tool paths like `python3 tools/upscale.py` will fail because `tools/` is relative. Always use:
```bash
cd /path/to/claude-code-video-toolkit && python3 tools/upscale.py ...
```
This is especially critical for background commands where the working directory may not be obvious.

### Tool Categories

| Type | Tools | When to Use |
|------|-------|-------------|
| **Project tools** | voiceover, music, music_gen, sfx, sync_timing | During video creation workflow |
| **Utility tools** | redub, addmusic, notebooklm_brand, locate_watermark | Quick transformations on existing videos |
| **Cloud GPU** | image_edit, upscale, dewatermark, sadtalker, qwen3_tts, music_gen, flux2 | AI processing via RunPod or Modal (`--cloud runpod\|modal`) |

Utility tools work on any video file without requiring a project structure.

### Voiceover Generation

```bash
# Per-scene generation (recommended)
python tools/voiceover.py --scene-dir public/audio/scenes --json


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [digitalsamba/claude-code-video-toolkit](https://github.com/digitalsamba/claude-code-video-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
