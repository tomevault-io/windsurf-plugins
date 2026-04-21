---
trigger: always_on
description: Always read .agents\rules\context-awareness.md
---

# CLAUDE.md
Always read .agents\rules\context-awareness.md
This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OWN (Only What's Needed) is a desktop application for video transcription and subtitle editing. It combines:
- A FastAPI backend server
- A PySide6 desktop GUI with system tray
- A web-based subtitle editor
- Support for Vosk and Whisper transcription engines

## Running the Application

```bash
# Activate virtual environment (if not already active)
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Run the application
python main.py
```

The application starts on port 5888 by default, opens a browser to the web interface, and launches the desktop tray app.

## Running Tests

```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_whisper_chunked.py

# Run with verbose output
pytest -v
```

## Architecture

### Directory Structure

```
stt_vosk/
├── core/           # Core transcription and processing logic
├── desktop/        # PySide6 desktop GUI and tray app
├── models/         # Data models (subtitles, styles)
├── server/         # FastAPI backend and database
├── web/            # Web frontend (HTML/CSS/JS)
├── tests/          # Test suite
└── data/           # Runtime data (uploads, thumbnails, exports)
```

### Key Components

**Server (`server/app.py`)**: FastAPI application providing REST API and WebSocket endpoints. Handles project CRUD, transcription tasks, model management, and file serving.

**Database (`server/database.py`)**: SQLite database with three main tables:
- `users`: User profile data
- `models`: Installed transcription models (Vosk/Whisper)
- `projects`: Video projects with subtitle data

**Model Manager (`server/model_manager.py`)**: Registry of available models and download handlers. Supports Vosk models (downloaded via URL) and Whisper models (downloaded via huggingface_hub).

**Transcription Engines**:
- `core/transcriber.py`: Vosk-based transcription with word-level timings
- `core/whisper_chunked.py`: Whisper transcription with chunked processing for RAM management
- `core/silence_detector.py`: Detects silence boundaries for chunking

**Subtitle System (`models/subtitle.py`)**: Data models for subtitles including:
- `WordTiming`: Single word with time span
- `StyledWord`: Word with optional style override and special marking
- `SubtitleSegment`: Group of styled words
- `SubtitleTrack`: Complete subtitle track with global style and special groups

**Desktop App (`desktop/`)**:
- `tray_app.py`: System tray icon with menu for model downloads
- `main_window.py`: PySide6 desktop window embedding web editor
- `setup.py`: First-time setup wizard

### Subtitle Data Model Hierarchy

The subtitle system uses a hierarchical model:

```
SubtitleTrack
├── global_style: SubtitleStyle
├── special_groups: dict[str, SpecialGroup]
├── segments: list[SubtitleSegment]
│   ├── style: SubtitleStyle
│   └── words: list[StyledWord]
│       ├── style_override: SubtitleStyle | null
│       ├── is_special: bool
│       └── group_id: str | null
```

**Special Words Feature**: Words can be marked as special (`is_special: true`) and optionally grouped (`group_id`). Special words can have individual style overrides or inherit from their group's style. This is used for highlighting specific words in the subtitle track.

### Transcription Flow

1. User uploads video via `/api/projects` POST
2. Server extracts thumbnail and generates timeline assets
3. User starts transcription via `/api/projects/{id}/transcribe` POST
4. Background task runs transcription (Vosk or Whisper)
5. Progress updates sent via WebSocket `/ws/progress/{task_id}`
6. Results stored as JSON in `projects.subtitle_data`

### Model Selection

Transcription engine and model are selected via POST body:
```json
{
  "engine": "vosk" | "whisper",
  "model": "model-name",
  "language": "hi" | "en" | ...
}
```

If no model is specified, the system falls back to:
- Vosk: Default model for the language or project root model
- Whisper: First installed Whisper model or auto-downloads `large-v3-turbo`

### Whisper Chunked Processing

Whisper transcription uses chunked processing to manage RAM:
1. Extract full audio to WAV
2. Detect silence boundaries
3. Split into ~30-second chunks at silence points
4. Load Whisper model once
5. Transcribe each chunk sequentially
6. Adjust timestamps by chunk offset
7. Merge all word timings

Configuration in `server/config.py`:
- `WHISPER_MAX_CHUNK_DURATION`: 30 seconds
- `WHISPER_MIN_SILENCE_DURATION`: 0.5 seconds
- `WHISPER_SILENCE_THRESHOLD`: -40 dB

### FFmpeg Integration

FFmpeg is used for:
- Audio extraction from video (16 kHz mono WAV)
- Thumbnail generation
- Video export with burned-in subtitles
- Timeline sprite sheet generation

FFmpeg commands run with `CREATE_NO_WINDOW` flag on Windows to avoid console popups.

### Web Frontend Architecture

The web editor is split into three main JavaScript modules:

**`web/js/editor.js`**: Main controller that wires together all components. Handles:
- Project loading and saving
- Word selection and special marking
- Style control initialization
- Tab switching and UI state management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/damameet14) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
