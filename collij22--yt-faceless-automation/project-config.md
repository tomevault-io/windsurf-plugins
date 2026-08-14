---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 🚀 Current Version: V4 Pipeline

**IMPORTANT**: Always use the V4 pipeline for production. Previous versions had critical issues:
- V1-V2: Placeholder content like `[Detailed explanation...]`
- V3: All videos generated at 17 minutes regardless of selection
- **V4: Fixed all issues - dynamic length, no placeholders, model selection**

## Commands

### Development Setup
```bash
# Windows: Create and activate venv
py -3.12 -m venv .venv
.venv\Scripts\activate
pip install -e .[dev]

# Configure environment
copy .env.example .env  # Edit with your directories and n8n webhook URLs
```

### Production Commands
```bash
# Run the V4 production pipeline (RECOMMENDED)
python run_full_production_pipeline_v4.py              # Default: sonnet model
python run_full_production_pipeline_v4.py --model claude   # Comprehensive content
python run_full_production_pipeline_v4.py --model haiku    # Concise, viral content
python run_full_production_pipeline_v4.py --model sonnet   # Balanced (default)

# Legacy commands (archived)
# The following legacy pipeline scripts are preserved in `archive/` for reference only:
# - run_full_production_pipeline.py (V1)
# - run_full_production_pipeline_v2.py (V2)
# - run_full_production_pipeline_v3.py (V3)
# - claude_script_generator.py / _v2.py / _v3.py
# Do not use these for production; V4 supersedes them.

# Testing
pytest
pytest tests/test_assembly.py -v  # Run specific test

# Code quality
black src/ tests/
isort src/ tests/
mypy src/
```

## Architecture Overview

This is a faceless YouTube automation system combining Claude subagents, MCP servers, n8n workflows, and Python orchestration.

### Core Components

1. **Subagents** (`.claude/agents/`): Specialized Claude agents for each phase
   - `research-analyst`: High-RPM niche discovery, trend validation, idea generation
   - `scriptwriter`: High-retention script generation with SSML markers, SEO metadata
   - `asset-curator`: B-roll and asset gathering via Firecrawl MCP
   - `voiceover-producer`: TTS production via n8n webhook
   - `video-assembler`: FFmpeg-based video assembly orchestration
   - `uploader`: YouTube upload via n8n
   - `optimizer`: Analytics-driven A/B testing and improvements
   - `revenue-analyst`: Monetization expansion strategies

2. **MCP Integration**: 
   - Firecrawl MCP for web scraping/search
   - n8n MCP for workflow automation
   - Ref MCP for latest documentation retrieval
   - Configuration in `docs/mcp-setup.md`

3. **Python Orchestrator** (`src/yt_faceless/`):
   - `cli.py`: Entry point with `ytfaceless` command
   - `orchestrator.py`: Main coordination logic
   - `assembly.py`: FFmpeg video assembly
   - `config.py`: Environment and configuration management
   - `youtube_metadata.py`: Metadata handling

4. **n8n Workflows** (`workflows/`):
   - `tts_webhook.json`: TTS processing webhook
   - `youtube_upload.json`: YouTube upload automation

### Content Pipeline

Phase 1 → Research (research-analyst) → `data/ideas/*.json`
Phase 2 → Script (scriptwriter) → `content/{slug}/script.md`, `metadata.json`  
Phase 3 → Assets + TTS (asset-curator, voiceover-producer) → `audio.wav`, `assets/`
Phase 4 → Assembly (video-assembler) → `final.mp4`
Phase 5 → Upload (uploader) → YouTube with SEO metadata
Phase 6 → Optimize (optimizer) → `reports/*.md` with experiments
Phase 7 → Monetize (revenue-analyst) → Revenue tracking and expansion

### Key Conventions

- Python 3.12+ required
- Black formatter (88 char line length)
- Conventional Commits for version control
- All subagents can access MCP tools unless explicitly restricted
- Batch TTS requests to minimize costs
- Use Firecrawl for summaries instead of full page scraping when possible

## V4 Pipeline Improvements

### Script Generation (`claude_script_generator_v4.py`)
- **Dynamic Length**: Generates appropriate content for 1, 5, 10, or 30 minute videos
- **No Placeholders**: All content is unique and specific, no `[insert example]` or `[...]`
- **Model Support**: Choose between claude (comprehensive), haiku (concise), or sonnet (balanced)
- **Accurate Timestamps**: END timestamps match actual video duration (e.g., `[END - 6:15]` for 6.25 min video)

### Bulletproof Video Production Features
- **Openverse/Wikimedia Compliant Clients**: Query sanitization, polite `User-Agent` (+ contact), Accept JSON; Commons downloads use `Referer`; 400/401/403/404 non‑retry; 429 honors Retry‑After; per‑host concurrency and delays
- **Thumbnail‑First Downloads**: Prefer `thumburl` for Commons to avoid hotlink 403s; headered direct downloads with validation
- **Smart Asset Deduplication**: Perceptual hashing with automatic URL-based fallback
- **Automatic Fallbacks**: Gradient cards generated and injected per scene when assets unavailable
- **Perfect Audio Sync**: FFprobe-based duration calculation (not word count estimates)
- **License Compliance**: LicenseValidator ensures commercial use rights and adds attribution
- **YouTube Compatibility**: Description length clamping to prevent API errors
- **Configurable FPS + Stills Hold**: Ken Burns respects FPS; still images are held for full scene duration via fps+tpad

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [collij22/yt-faceless-automation](https://github.com/collij22/yt-faceless-automation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
