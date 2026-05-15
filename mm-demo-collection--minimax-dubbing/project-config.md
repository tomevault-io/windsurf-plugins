---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# minimax_dubbing

A Vue 3 + Django AI-powered video dubbing system with multi-modal speaker recognition, automatic subtitle generation (ASR), voice separation, translation, and text-to-speech synthesis.

## 🚀 Development Commands

### ⚡ Quick Setup (一键初始化)
```bash
# 快速初始化整个系统
./init.sh

# 或者手动初始化
python manage.py init_system

# 或者更细粒度的控制
python manage.py init_admin --username admin --password admin123
```

### Backend (Django)
```bash
# Install dependencies
pip install -r requirements.txt

# Database operations
python manage.py migrate                        # Apply all migrations (REQUIRED!)
python manage.py makemigrations                 # Create new migrations
python manage.py showmigrations                 # Show migration status

# System initialization
python manage.py init_system                    # Complete system init (creates admin)
python manage.py init_admin                     # Create admin user only
python manage.py init_admin --force             # Force recreate admin user

# Fix incomplete user registrations (missing UserConfig)
python fix_incomplete_users.py

# Development server
python manage.py runserver 0.0.0.0:5172

# Testing
python manage.py test
python manage.py test app_name                  # Test specific app
python manage.py test app_name.tests.TestClassName  # Test specific class
python manage.py test --verbosity=2             # Verbose test output

# Database shell and inspection
python manage.py shell                          # Django shell
python manage.py dbshell                        # Database SQL shell

# Check configuration validity
python check_config.py
```

### Frontend (Vue 3 + Vite)
```bash
# Install dependencies
cd frontend && npm install

# Development server
npm run dev  # Runs on port 5173

# Build for production
npm run build

# Type checking
vue-tsc -b

# Preview production build
npm run preview
```

### Required Dual Server Setup
**Critical**: Both backend (port 5172) and frontend (port 5173) must run simultaneously for the application to work.

```bash
# Terminal 1 - Backend
python manage.py runserver 0.0.0.0:5172

# Terminal 2 - Frontend
cd frontend && npm run dev
```

## 🏗️ Architecture Overview

### Technology Stack
- **Frontend**: Vue 3 + TypeScript + Element Plus + Vite + Pinia
- **Backend**: Django 5.2.6 + Django REST Framework
- **Database**: SQLite (development) / PostgreSQL (production)
- **AI Integration**:
  - MiniMax API - Translation and TTS
  - Qwen-VL (DashScope) - Visual-language model for speaker naming
  - Qwen LLM (DashScope) - Speaker-subtitle assignment
  - Alibaba Cloud NLS - ASR (Automatic Speech Recognition)
- **Computer Vision**: FaceNet + MTCNN (face detection) + DBSCAN (clustering)
- **Audio Processing**: Demucs (vocal separation), PyDub, librosa, FFmpeg

### Application Structure
```
/
├── backend/                    # Django settings and core configuration
├── authentication/             # Custom user auth with API key system
│   ├── management/commands/    # init_system, init_admin commands
│   └── models.py               # User and UserConfig models
├── projects/                   # Project management (main entity)
├── segments/                   # Translation segments with timestamps
├── speakers/                   # Speaker recognition and diarization (NEW)
│   └── models.py               # SpeakerRecognitionTask, Speaker models
├── services/                   # Business logic and AI integrations
│   ├── algorithms/             # Timestamp alignment algorithms
│   ├── business/               # Core business services
│   ├── clients/                # External API clients (MiniMax)
│   ├── parsers/                # SRT/subtitle file parsers
│   ├── asr/                    # ASR (Automatic Speech Recognition) (NEW)
│   ├── audio_separator/        # Demucs vocal separation (NEW)
│   └── speaker_diarization/    # Multi-modal speaker pipeline (NEW)
│       ├── face_detector.py    # FaceNet + MTCNN face detection
│       ├── clusterer.py        # DBSCAN face clustering
│       ├── vlm_naming.py       # Qwen-VL speaker profile generation
│       ├── llm_assignment.py   # Qwen LLM subtitle-speaker assignment
│       └── pipeline.py         # End-to-end orchestration
├── system_monitor/             # Background task monitoring
├── logs/                       # Centralized logging system
├── voices/                     # Voice management for TTS
├── voice_cloning/              # Voice cloning features
└── frontend/src/
    ├── components/
    │   ├── editor/             # Inline editing components
    │   ├── audio/              # Audio playback and visualization
    │   ├── project/            # Project management UI
    │   ├── voice/              # Voice/speaker management
    │   └── speakers/           # Speaker recognition UI (NEW)
    ├── composables/            # Vue 3 composition functions
    │   ├── useInlineEditor.ts  # Core editing logic with debounced saves
    │   ├── useSegmentSelection.ts  # Multi-select operations
    │   ├── useSegmentValidation.ts # Real-time validation
    │   ├── useSegmentBatch.ts  # Batch operations (translate/TTS)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mm-demo-collection/minimax_dubbing](https://github.com/mm-demo-collection/minimax_dubbing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
