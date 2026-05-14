---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a YouTube Shorts Creator project - a production-ready tool for converting long YouTube videos into engaging YouTube shorts with AI-powered content analysis and professional video effects.

## Development Environment

- **Python Version**: 3.13 (specified in `.python-version`)
- **Package Manager**: uv (modern Python package manager)
- **Project Structure**: Standard Python package with `pyproject.toml` configuration

## Key Commands

### Package Management
```bash
# Install dependencies
uv sync

# Add new dependency
uv add <package-name>

# Remove dependency
uv remove <package-name>

# Install as global tool
uv tool install -e .
```

### Running the Application
```bash
# After installation as global tool
shorts-creator -v /path/to/video.mp4

# Or run directly with uv
uv run python -m shorts_creator.main -v /path/to/video.mp4
```

## Project Architecture

### Directory Structure
```
src/
└── shorts_creator/
    ├── domain/
    │   └── models.py         # Pydantic models (Speech, YouTubeShort, etc.)
    ├── pipeline/
    │   ├── audio_retriever.py    # Audio extraction from video
    │   ├── speech_to_text.py     # faster-whisper transcription
    │   ├── shorts_generator.py   # AI-powered content analysis with duration enforcement
    │   ├── video_cutter.py       # Video segment extraction
    │   ├── caption_generator.py  # ASS subtitle generation
    │   ├── ass_generator.py      # Advanced subtitle styling
    │   └── storage.py            # File I/O utilities
    ├── video_effect/
    │   ├── video_effect.py       # Video effect classes (Audio, Text, Captions, etc.)
    │   ├── video_effect_service.py # Effect application service
    │   └── strategies.py         # Effect strategy patterns (BASIC implemented)
    ├── assets/
    │   └── fonts.py              # Embedded font resources
    ├── settings/
    │   └── settings.py           # Configuration management with CLI parsing
    └── main.py                   # Main entry point with progress tracking
poc/                              # Proof-of-concept experiments (not production code)
```

### Current Implementation
- **Full Pipeline**: Audio extraction → Transcription → AI analysis → Video cutting → Effects application
- **AI Integration**: OpenAI/OpenRouter for content analysis with strict duration enforcement (≥minimum, up to 20-30% longer)
- **Video Effects**: Professional ffmpeg-based video enhancement pipeline with multiple strategies
- **Caption System**: Advanced ASS subtitle generation with precise timing and styling
- **Console Script**: Global `shorts-creator` command via uv tool install with comprehensive CLI options
- **Progress Tracking**: Real-time progress bars for all pipeline operations with detailed status
- **Configuration**: Environment variables and command-line arguments with debug mode support

### Key Technologies
- **faster-whisper**: High-accuracy speech transcription with timestamp precision
- **OpenAI API**: Content analysis and YouTube metadata generation with structured responses
- **FFmpeg**: Professional video processing and encoding via ffmpeg-python
- **MoviePy**: Video manipulation and effects application
- **OpenCV**: Computer vision for video analysis and processing
- **Pydantic**: Type-safe configuration and data models with settings management
- **pysubs2**: Advanced subtitle processing and ASS format support
- **tqdm**: Progress tracking and user feedback with dynamic progress bars

## Code Conventions
- Standard Python package structure with domain/pipeline separation
- Pydantic models for all data structures and configuration
- Logging configured at INFO level with timestamp formatting (DEBUG mode available)
- Module-level logger instances using `__name__`
- Type hints and modern Python 3.13 features
- Environment-based configuration with .env support
- Comprehensive error handling and progress tracking

## Recent Updates

### Duration Enforcement (Latest)
- **Strict Minimum Duration**: AI now enforces minimum duration requirements (≥ specified seconds)
- **Flexible Maximum**: Allows 20-30% overage but never shorter than minimum
- **Smart Content Combination**: AI can combine adjacent segments to meet duration requirements
- **Updated Prompts**: Enhanced system prompts with clear duration constraints and instructions

### Video Effects Pipeline
- **Strategy Pattern**: Implemented configurable video effects strategies
- **BASIC Strategy**: Complete pipeline with audio normalization, format conversion, text overlays, captions, and encoding
- **Extensible Design**: Easy to add new effect strategies by extending VideoEffectsStrategy enum

# YouTube Shorts Clip Effects — Requirements (Software Engineering Channel)

This document defines **must/should** requirements for producing fast, legible, and rewatchable YouTube Shorts tailored to software‑engineering topics (code, CLI, APIs). Treat it as a checklist for scripting, editing, and QC.

---

## 1) Core Objectives

* **Clarity first:** Make the key idea understandable on a 6–6.7″ phone screen in noisy environments.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vitalii-honchar/youtube-shorts-creator](https://github.com/vitalii-honchar/youtube-shorts-creator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
