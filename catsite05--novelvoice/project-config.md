---
trigger: always_on
description: This file provides guidance to Qoder (qoder.com) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Qoder (qoder.com) when working with code in this repository.

## Project Overview
NovelVoice is a Flask-based web application that converts Chinese novel text files into audio books with character-aware voice synthesis. The system uses LLM to analyze text, identify characters and dialogue, then generates multi-voice audio using Azure TTS (via EasyVoice).

## Architecture

### Core Components
All Python modules are located in the `app/` directory:

1. **Flask App (`app/app.py`)**: Entry point that initializes the Flask application, database, authentication, and routes
2. **Database Models (`app/models.py`)**: SQLAlchemy models for User, Novel, Chapter, and Character entities
3. **Configuration (`app/config.py`)**: Application configuration including session settings and database paths
4. **File Upload (`app/upload.py`)**: Handles novel file uploads and creates Novel records
5. **Chapter Processing (`app/chapter.py`)**: Novel parsing and chapter segmentation logic
6. **Audio API (`app/audio.py`)**: HTTP endpoints for audio playback and streaming
7. **Audio Generator (`app/audio_generator.py`)**: Core audio generation engine with multi-threaded pipeline (script generation → TTS synthesis → file writing)
8. **LLM Integration (`app/llm_client.py`)**: Connects to LLM API for voice script generation
9. **Voice Script Generation (`app/voice_script.py`)**: Converts LLM output to TTS-compatible format
10. **TTS Clients**:
    - **EasyVoice Client (`app/easyvoice_client.py`)**: Interfaces with local EasyVoice service (Azure TTS proxy)
    - **Edge TTS Client (`app/edgetts_client.py`)**: Direct Edge TTS integration with streaming support (default)
11. **HLS Manager (`app/hls_manager.py`)**: FFmpeg-based MP3→HLS conversion for adaptive streaming

### Data Flow
1. User uploads a `.txt` novel file → `app/upload.py` saves it and creates Novel record
2. `split_novel_into_chapters()` parses the file using regex patterns to detect Chinese chapter markers (第X章, 序章, etc.)
3. When user plays a chapter → `app/audio.py` HTTP endpoints handle the request
4. **Audio Generation Pipeline** (`app/audio_generator.py`):
   - **Thread 1 (Script Generator)**: Splits content into ~1500 char segments → sends to LLM → converts to voice script → puts in queue
   - **Thread 2 (Audio Producer)**: Consumes script queue → calls TTS (Edge TTS default, or EasyVoice if `USE_EASYVOICE=1`) → writes MP3 chunks to file
   - **Thread 3 (HLS Converter)**: Monitors MP3 file growth → incrementally converts to HLS using FFmpeg (enables adaptive streaming)
5. `app/voice_script.py` converts LLM JSON output to TTS format, mapping characters to Azure TTS voices from `voice.json`
6. Generated MP3 and HLS files are cached and served to the client
7. **HLS Streaming** (`app/hls_manager.py`): 
   - Converts MP3→HLS (playlist.m3u8 + MPEG-TS segments) using FFmpeg
   - Supports incremental conversion: while MP3 is still being generated, HLS segments are created in real-time
   - Client can start playback before full MP3 generation completes

### Database Schema
- **User**: User authentication (username, password_hash, is_superuser)
- **Novel**: Stores uploaded novel metadata (title, author, file_path, upload_date, user_id FK)
- **Chapter**: Contains chapter metadata (title, start_position in file, audio_file_path, audio_status, novel_id FK)
- **Character**: Caches character info extracted by LLM (name, gender, personality, voice mapping)

### Voice Configuration
`voice.json` maps character traits to Azure TTS voice IDs:
- Gender: Male/Female
- Personalities: Warm, Lively, Passion, Sunshine, Professional, Reliable, Humorous, Bright
- Default narrator voice: zh-CN-YunxiNeural

## Development Commands

### Running the Application
```bash
source ./venv/bin/activate
python3 app/app.py
```
Application runs at http://localhost:5002 in debug mode.

For quick testing with environment variables set:
```bash
./test.sh
```

For deployment using Docker:
```bash
./deploy.sh
```

### Environment Variables
Required environment variables (see `test.sh` for example):
- `LLM_API_KEY`: API key for LLM service
- `LLM_BASE_URL`: Base URL for LLM API endpoint
- `LLM_MODEL`: Model identifier (e.g., "qwen3-max")
- `USE_EASYVOICE`: Set to `1` to use EasyVoice instead of Edge TTS (default: Edge TTS)
- `EASYVOICE_BASE_URL`: URL of local EasyVoice TTS service (optional, only needed if `USE_EASYVOICE=1`)
- `SECRET_KEY`: Flask secret key for session management

### Database Initialization
Database is automatically created on first run via `db.create_all()` in `app/app.py`.
SQLite database file: `instance/novelvoice.db`

### Dependencies
Install via:
```bash
pip install -r requirements.txt
```
Core dependencies: Flask 2.3.3, Flask-SQLAlchemy 3.0.5, requests 2.31.0, edge-tts 6.1.18, ffmpeg-python 0.2.0

**System Dependencies:**
- FFmpeg (required for HLS streaming): `sudo apt-get install ffmpeg` (Ubuntu/Debian) or `brew install ffmpeg` (macOS)

### Testing
Test HLS functionality:
```bash
python3 test_hls.py
```
This verifies FFmpeg installation and HLS conversion for an existing chapter audio file.

## Key Implementation Details

### Authentication System
- Session-based authentication using Flask sessions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [catsite05/novelvoice](https://github.com/catsite05/novelvoice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
