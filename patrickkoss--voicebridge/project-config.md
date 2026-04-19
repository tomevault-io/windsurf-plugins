---
trigger: always_on
description: This file provides guidance to Claude Code when working with the VoiceBridge project.
---

# CLAUDE.md - VoiceBridge Project

This file provides guidance to Claude Code when working with the VoiceBridge project.

## Project Overview

VoiceBridge is a comprehensive bidirectional voice-text CLI tool that bridges speech and text seamlessly. Built on OpenAI's Whisper for speech recognition and VibeVoice for text-to-speech synthesis, with advanced features including:

### Core Features
- **Speech-to-Text (STT)**: Real-time transcription, file processing, batch operations
- **Text-to-Speech (TTS)**: High-quality voice synthesis with custom voices
- **GPU acceleration** (CUDA/Metal) with automatic device detection
- **Memory optimization** and streaming for large audio files  
- **Resume capability** for interrupted transcriptions
- **Performance monitoring** and session management
- **Audio processing**: noise reduction, normalization, splitting, enhancement
- **Export formats**: JSON, SRT, VTT, plain text, CSV
- **Hotkey support** for hands-free operation
- **Hexagonal architecture** with ports and adapters pattern

## Development Setup

### Virtual Environment & Dependencies

**IMPORTANT**: This project uses `uv` for fast Python package management and a virtual environment at `.venv/`. Always use the Makefile commands or `uv run` for operations.

```bash
# Initialize environment and install all dependencies
make prepare

# CUDA support (for GPU acceleration)
make prepare-cuda

# System tray support (optional)
make prepare-tray

# Manual uv setup (if needed)
uv venv .venv
uv pip install --editable ".[dev]"
```

### Key Commands

```bash
make help         # Show all available commands
make prepare      # Initialize .venv with uv and install dependencies
make prepare-cuda # Initialize .venv with CUDA support
make prepare-tray # Initialize .venv with system tray support
make lint         # Run ruff linting and auto-fix issues
make test         # Run all tests with coverage report
make test-fast    # Run tests without coverage
make clean        # Clean up cache files and .venv
```

### Manual Commands (using uv directly)

```bash
# Always use uv run for any manual commands
uv run ruff check --fix .          # Linting
uv run pytest                      # Testing  
uv run python -m voicebridge --help # Run CLI
uv pip install package-name        # Install packages
```

## Architecture

### Directory Structure
```
voicebridge/
├── domain/          # Core business logic and models
│   └── models.py    # Data models (WhisperConfig, TTSConfig, GPUInfo, etc.)
├── ports/           # Interfaces/abstract base classes  
│   └── interfaces.py
├── adapters/        # External integrations
│   ├── audio/       # Audio recording, playback, processing
│   ├── system.py    # GPU detection, memory monitoring
│   ├── transcription.py     # Whisper service implementation
│   ├── vibevoice_tts.py     # VibeVoice TTS implementation
│   ├── session.py   # Session persistence
│   └── config.py    # Configuration management
├── services/        # Application services
│   ├── transcription_service.py  # STT orchestration
│   ├── tts_service.py           # TTS orchestration and daemon
│   ├── performance_service.py   # Performance monitoring
│   ├── batch_service.py         # Batch processing
│   ├── export_service.py        # Export functionality
│   ├── confidence_service.py    # Quality analysis
│   └── resume_service.py        # Resume functionality
├── cli/             # Command line interface
│   ├── app.py       # Main CLI app with Typer
│   └── commands.py  # Command implementations
└── tests/          # Test suite
```

### Key Features Implemented

#### Speech-to-Text (STT)
1. **Real-time Transcription**: Hotkey-driven live speech recognition
2. **File Processing**: Support for MP3, WAV, M4A, FLAC, OGG formats
3. **Batch Processing**: Directory-wide transcription with parallel workers
4. **GPU Acceleration**: Automatic detection and selection of CUDA/Metal devices
5. **Memory Optimization**: Chunked processing and memory limit enforcement
6. **Resume Capability**: Session persistence for interrupted transcriptions
7. **Export Formats**: JSON, SRT, VTT, plain text, CSV output

#### Text-to-Speech (TTS)
1. **VibeVoice Integration**: High-quality neural voice synthesis
2. **Multiple Input Modes**: Clipboard monitoring, text selection, direct input
3. **Custom Voices**: Voice sample detection and management
4. **Streaming/Non-streaming**: Real-time or complete generation modes
5. **Hotkey Controls**: Global shortcuts for hands-free operation
6. **Audio Output**: Play immediately, save to file, or both

#### Advanced Processing
1. **Audio Enhancement**: Noise reduction, normalization, silence trimming
2. **Audio Splitting**: Duration, silence, or size-based segmentation
3. **Confidence Analysis**: Quality assessment and review flagging
4. **Performance Monitoring**: Comprehensive metrics collection and reporting
5. **Session Management**: Progress tracking and resume functionality
6. **Profile Management**: Multiple configuration profiles
7. **Webhook Integration**: External notification support

## Code Standards

- **Architecture**: Hexagonal/Ports & Adapters pattern
- **Python Version**: 3.10+
- **Linting**: ruff with auto-fix
- **Testing**: pytest with coverage reporting

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PatrickKoss) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
