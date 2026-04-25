---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

VoiceMoodAnalyzer is a multi-stage AI pipeline that analyzes emotional state from voice recordings by combining three signals:
1. Audio transcription (Whisper.cpp - local, small model ~466MB)
   - No internet required for transcription
   - 6x realtime speed, excellent accuracy
2. Audio emotion detection (Wav2Vec2 model - local, 97.5% accuracy)
   - Model: r-f/wav2vec-english-speech-emotion-recognition
   - 7 emotions: angry, disgust, fear, happy, neutral, sad, surprise
3. Text sentiment analysis (DistilRoBERTa model - local)
4. Emotion fusion via a database-driven matrix lookup

The system is fully containerized with Docker and designed for Azure VM deployment with mobile-friendly web access. All AI models run locally with no external API dependencies.

## Development Commands

### Docker Environment (Primary Development Method)

```bash
# Start all services (backend, frontend, postgres)
docker-compose up -d --build

# View logs from all containers
docker-compose logs -f

# View logs from specific service
docker-compose logs -f backend
docker-compose logs -f frontend
docker-compose logs -f postgres

# Restart specific service
docker-compose restart backend

# Stop all services
docker-compose down

# Stop and remove volumes (full reset including database)
docker-compose down -v

# Rebuild after code changes
docker-compose up -d --build
```

### Local Development (Without Docker)

**Backend:**
```bash
cd backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Requires PostgreSQL running separately
# Models will download on first run (~2GB)
uvicorn app:app --reload --host 0.0.0.0 --port 8000
```

**Frontend:**
```bash
cd frontend
npm install
npm run dev          # Development server on port 3000
npm run build        # Production build
npm run preview      # Preview production build
```

### Database Operations

```bash
# Access PostgreSQL shell
docker-compose exec postgres psql -U postgres -d mito_books

# Backup database
docker-compose exec postgres pg_dump -U postgres mito_books > backup.sql

# Restore database
docker-compose exec -T postgres psql -U postgres mito_books < backup.sql

# Reset database (re-run init scripts)
docker-compose down -v && docker-compose up -d
```

### Testing

```bash
# Test API endpoints
./test_api.sh

# Manual API test
curl http://localhost:8000/
curl http://localhost:8000/api/matrix
curl -X POST http://localhost:8000/api/analyze -F "file=@test.mp3"
```

## Architecture & Data Flow

### Request Pipeline (POST /api/analyze)

The analysis pipeline in `backend/app.py` follows this exact sequence:

1. **Upload & Validation** (`app.py:analyze_voice`)
   - Validate file size (<25MB) and format (.wav, .mp3, .m4a, .ogg, .flac, .webm)
   - Save to temporary file

2. **Whisper Transcription** (`services/whisper_cpp_service.py`)
   - Local transcription using whisper.cpp (small model)
   - No internet required, 6x realtime speed
   - Returns: `transcribed_text: str`

3. **Audio Emotion Detection** (`services/audio_emotion.py`)
   - **Duration-based processing**: Only runs for recordings ≤15 seconds
   - For recordings >15 seconds: Skipped (defaults to "neutral" with 0.0 confidence) to save processing time (~15-20s per analysis)
   - Load audio, resample to 16kHz mono
   - Run through Wav2Vec2 model (`r-f/wav2vec-english-speech-emotion-recognition`)
   - **97.5% accuracy** - Fine-tuned on 4,720 samples from SAVEE, RAVDESS, and TESS datasets
   - Returns: `audio_emotion: str` (angry/disgust/fear/happy/neutral/sad/surprise), `audio_confidence: float`

4. **Text Emotion Detection** (`services/text_emotion.py`)
   - Tokenize transcribed text
   - Run through DistilRoBERTa model (`j-hartmann/emotion-english-distilroberta-base`)
   - Returns: `text_emotion: str` (neutral/happy/sad/angry/surprised/fearful/disgusted), `text_confidence: float`

5. **Emotion Fusion** (`services/fusion_service.py`)
   - Database lookup in `voice_matrix` table using (audio_emotion, text_emotion) as composite key
   - Fallback strategy: neutral+neutral → ultimate fallback: "Unknown"
   - Returns: `{final_mood, emoji, description}`

6. **Database Persistence** (`models/voice_analysis.py`)
   - Save complete analysis to `voice_analysis` table
   - Includes all intermediate results + final mood

### ML Model Lifecycle

**Singleton Pattern for Models** (important for memory efficiency):
- `get_whisper_service()` in `whisper_cpp_service.py` - loads Whisper.cpp once, caches globally (lazy loaded)
- `get_audio_emotion_service()` in `audio_emotion.py` - loads Wav2Vec2 once, caches globally
- `get_text_emotion_service()` in `text_emotion.py` - loads DistilRoBERTa once, caches globally
- Hugging Face models preload on app startup via `@app.on_event("startup")` in `app.py:41-48`
- Whisper.cpp model loads on first transcription request (lazy loading)

**First Run Behavior**:
- Hugging Face models download to Docker container (~2GB)
- Whisper.cpp small model downloads on first use (~466MB)
- Total model size: ~2.5GB
- Initial build takes 10-15 minutes, subsequent starts <30 seconds

### Database Schema Details


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/orosales) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
