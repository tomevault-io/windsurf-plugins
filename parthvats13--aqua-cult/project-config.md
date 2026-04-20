---
trigger: always_on
description: **Type:** Aquaculture Management System
---

# AquaSense - Project Context for AI Assistants

## Project Overview

**Name:** AquaSense
**Type:** Aquaculture Management System
**Platform:** Android (Kotlin/Jetpack Compose) + Python Backend (FastAPI)
**Purpose:** Help fish farmers manage tanks, detect diseases, get AI recommendations, and purchase supplies
**Development Stage:** Initial implementation (backend first approach)

---

## Quick Start

### Running the Backend
```bash
cd aquasense_backend
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
export GEMINI_API_KEY="your-key-here"
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

### Running the Android App
1. Open project in Android Studio
2. Update `NetworkModule.kt`:
   - Emulator: `BASE_URL = "http://10.0.2.2:8000/api/v1/"`
   - Physical device: `BASE_URL = "http://<your-local-ip>:8000/api/v1/"`
3. Run on emulator or device

---

## Technology Stack

### Backend (Python)
```
FastAPI          - Web framework
Uvicorn          - ASGI server
SQLAlchemy       - ORM for SQLite
Pydantic         - Data validation
google-generativeai - Gemini API
TensorFlow       - ML model inference (.keras)
gTTS             - Text-to-speech
python-multipart - File uploads
```

### Frontend (Android)
```
Kotlin 2.0       - Language
Jetpack Compose  - UI framework
Hilt             - Dependency injection
Retrofit         - REST client
OkHttp           - HTTP + WebSocket
Room             - Local database
CameraX          - Camera API
ExoPlayer        - Audio playback
```

---

## Key Architectural Decisions

### 1. No LangChain/LangGraph
**Decision:** Use Direct Gemini API instead
**Reason:** Simpler, fewer dependencies, easier to debug
**Trade-off:** Less abstraction, manual prompt management

### 2. Server-side ML
**Decision:** Disease detection runs on Python backend
**Reason:** ML team provides .keras model, easier to update
**Trade-off:** Requires network call, but images are small

### 3. On-device STT
**Decision:** Android SpeechRecognizer instead of cloud STT
**Reason:** Free, fast, works offline
**Trade-off:** Less accurate than Google Cloud STT, but sufficient

### 4. SQLite Database
**Decision:** SQLite instead of PostgreSQL
**Reason:** Simpler setup for local development
**Trade-off:** No advanced features, but not needed for demo

### 5. No Authentication
**Decision:** Single hardcoded user for local development
**Reason:** Simplifies development, not deploying to production
**Trade-off:** Not production-ready, but can add later

---

## File Structure Conventions

### Backend Structure
```
aquasense_backend/
├── main.py                      # FastAPI app entry point
├── requirements.txt             # Python dependencies
├── .env                         # Environment variables (not committed)
├── config/
│   ├── __init__.py
│   ├── settings.py              # Pydantic settings
│   └── database.py              # SQLAlchemy setup
├── models/                      # SQLAlchemy ORM models
│   ├── __init__.py
│   ├── user.py
│   ├── tank.py
│   ├── water_quality.py
│   ├── product.py
│   └── order.py
├── schemas/                     # Pydantic schemas (request/response)
│   ├── __init__.py
│   ├── tank.py
│   ├── analysis.py
│   └── voice.py
├── api/
│   └── v1/
│       ├── __init__.py
│       ├── router.py            # API router
│       └── endpoints/
│           ├── tanks.py
│           ├── products.py
│           ├── analysis.py
│           └── voice_agent.py   # WebSocket endpoint
├── services/                    # Business logic
│   ├── __init__.py
│   ├── tank_service.py
│   ├── analysis_service.py
│   └── voice_service.py
├── ai/                          # AI/Gemini integration
│   ├── __init__.py
│   ├── gemini_client.py
│   ├── prompts.py
│   └── session_memory.py
├── ml/                          # ML model
│   ├── __init__.py
│   ├── disease_classifier.py
│   ├── preprocessing.py
│   └── models/
│       └── fish_disease.keras   # Provided by ML team
├── knowledge/                   # Aquaculture knowledge base
│   ├── treatments.json
│   ├── diseases.json
│   └── species_info.json
├── websocket/                   # WebSocket handlers
│   ├── __init__.py
│   ├── handler.py
│   └── message_types.py
└── tests/
```

### Android Structure (Clean Architecture)
```
app/src/main/java/com/parth/aquasense/
├── AquaSenseApp.kt              # Application class
├── MainActivity.kt              # Single activity
├── di/                          # Hilt modules
│   ├── AppModule.kt
│   ├── NetworkModule.kt
│   └── DatabaseModule.kt
├── data/                        # Data layer
│   ├── local/
│   │   ├── dao/
│   │   ├── entity/
│   │   └── AquaSenseDatabase.kt
│   ├── remote/
│   │   ├── api/
│   │   │   ├── TankApi.kt
│   │   │   ├── ProductApi.kt
│   │   │   └── AnalysisApi.kt
│   │   └── websocket/
│   │       └── VoiceAgentWebSocket.kt
│   └── repository/
│       ├── TankRepository.kt
│       └── ProductRepository.kt
├── domain/                      # Domain layer
│   ├── model/
│   │   ├── Tank.kt
│   │   ├── WaterQuality.kt
│   │   └── Product.kt
│   └── usecase/
│       ├── GetTanksUseCase.kt
│       └── AnalyzeTankUseCase.kt
├── presentation/                # UI layer
│   ├── navigation/
│   │   ├── NavGraph.kt
│   │   └── Screen.kt
│   ├── theme/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Parthvats13) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
