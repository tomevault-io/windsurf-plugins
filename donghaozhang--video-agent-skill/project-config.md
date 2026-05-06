---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Git Workflow Commands

**Always run after each successful implementation:**
```bash
git add .
git commit -m "descriptive commit message"
git push origin main
```

## Project Overview

This is the **AI Content Pipeline** - a unified Python package for multi-modal AI content generation.

- **Unified Interface**: Single package with console commands `ai-content-pipeline` and `aicp`
- **73 AI models across 12 categories** — see [Models Reference](docs/reference/models.md)
- **Central Model Registry**: Single source of truth in `registry.py` + `registry_data.py`
- **Click-based CLI**: All commands use Click framework, vimax available as `aicp vimax` subgroup
- **Unix-style flags**: `--json`, `--quiet`, `--stream`, `--input` for scripting and CI
- **YAML Configuration**: Multi-step content generation workflows
- **Parallel Execution**: 2-3x speedup with `PIPELINE_PARALLEL_ENABLED=true`

## Environment Setup

### Python Virtual Environment (Required)
```bash
# Create and activate virtual environment (run from project root)
python3 -m venv venv
source venv/bin/activate  # Linux/Mac
# or venv\Scripts\activate  # Windows

# Install all dependencies from root
pip install -e .
# or: pip install -r requirements.txt
```

**Memory**: Virtual environment at project root `venv/`. Always activate before running scripts (`venv\Scripts\activate` on Windows, `source venv/bin/activate` on Linux/Mac).

## Common Commands

### CLI Commands
```bash
# List available AI models
aicp list-models

# Generate single image
aicp generate-image --text "A beautiful sunset" --model flux_dev

# Create video from text (text -> image -> video)
aicp create-video --text "A beautiful sunset"

# Run pipeline from YAML config
aicp run-chain --config input/pipelines/config.yaml

# Run with parallel execution (2-3x speedup)
PIPELINE_PARALLEL_ENABLED=true aicp run-chain --config config.yaml

# Analyze video with AI (Gemini 3 Pro via FAL)
aicp analyze-video -i video.mp4

# Generate avatar with lip-sync
aicp generate-avatar --image-url "https://..." --audio-url "https://..."

# Transcribe audio
aicp transcribe --input audio.mp3

# Generate image grid
aicp generate-grid --text "mountain landscape" --layout 2x2

# Upscale image
aicp upscale-image --image photo.png --upscale 2

# Transfer motion from video to image
aicp transfer-motion --image-url "https://..." --video-url "https://..."

# Novel-to-video pipeline (vimax subgroup)
aicp vimax idea2video --idea "A samurai's journey at sunrise"
aicp vimax novel2movie --novel novel.txt --storyboard-only

# JSON output for scripting
aicp list-models --json | jq '.text_to_video[]'
```

See [CLI Reference](docs/reference/cli-commands.md) for full command documentation.

### Testing Commands
```bash
# Run full pytest suite (~844 tests)
python -m pytest tests/ -v

# Registry tests only
python -m pytest tests/test_registry.py tests/test_registry_data.py tests/test_auto_discovery.py -v

# Click CLI tests
python -m pytest tests/test_click_app.py tests/test_main_cli_flags.py -v

# Quick smoke tests
python tests/test_core.py

# Validate registry consistency
python scripts/validate_registry.py
```

## Architecture

### Package Structure
```
ai-content-pipeline/
├── packages/
│   ├── core/
│   │   ├── ai_content_pipeline/     # Main pipeline + central registry + Click CLI
│   │   │   ├── registry.py          # ModelDefinition + ModelRegistry (source of truth)
│   │   │   ├── registry_data.py     # All 73 model registrations
│   │   │   ├── _version.py          # Single source of truth for version
│   │   │   └── cli/                 # Click CLI (click_app.py + commands/)
│   │   └── ai_content_platform/     # Platform framework + vimax subgroup (aicp vimax)
│   ├── providers/
│   │   ├── google/veo/              # Google Veo integration
│   │   └── fal/                     # FAL AI services
│   │       ├── text-to-image/       # Image generation (8 models)
│   │       ├── image-to-image/      # Image transformation (8 models)
│   │       ├── text-to-video/       # Video generation (10 models)
│   │       ├── image-to-video/      # Image-to-video (15 models)
│   │       ├── video-to-video/      # Video-to-video (4 models)
│   │       ├── avatar-generation/   # Avatar creation (10 models)
│   │       └── speech-to-text/      # Speech transcription (1 model)
│   └── services/
│       ├── text-to-speech/          # ElevenLabs TTS
│       └── video-tools/             # Video processing
├── scripts/                         # Validation scripts
├── input/                           # Configuration files
├── output/                          # Generated content
├── tests/                           # Test suites (pytest, ~844 tests)
├── docs/                            # Documentation (see docs/index.md)
└── setup.py                         # Package installation
```

See [Architecture Overview](docs/reference/architecture.md) and [Package Structure](docs/reference/package-structure.md) for details.

### Key Architecture Points

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [donghaozhang/video-agent-skill](https://github.com/donghaozhang/video-agent-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
