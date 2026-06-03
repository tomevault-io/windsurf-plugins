---
trigger: always_on
description: This is an AI Agent Debate System (AI辩论代理系统) built on the AutoGen framework. It supports structured debates on customizable topics using multi-agent interactions with DeepSeek LLM and SiliconFlow speech services.
---

# AI Agent Debate System - Agent Guide

## Project Overview

This is an AI Agent Debate System (AI辩论代理系统) built on the AutoGen framework. It supports structured debates on customizable topics using multi-agent interactions with DeepSeek LLM and SiliconFlow speech services.

**Key Capabilities:**
- Multi-agent debate with pro, con, and judge roles
- Real-time WebSocket streaming of debate content
- Text-to-speech synthesis (SiliconFlow primary, Huawei Cloud backup)
- Auto-generation of debates from news URLs
- AI-generated debate posters (Volcano Engine)
- Vue.js frontend with mobile support
- Complete user system (login, profile)

## Technology Stack

### Backend
- **Python 3.9+** - Main language
- **AutoGen 0.9** - Multi-agent conversation framework
- **Flask** - REST API server (port 9000)
- **FastAPI + Uvicorn** - WebSocket/Streaming API (port 8001)
- **OpenAI SDK** - LLM API client
- **aiohttp** - Async HTTP client for speech APIs
- **BeautifulSoup** - Web scraping for news extraction

### Frontend
- **Vue.js 3** - Frontend framework
- **Vite 6.x** - Build tool
- **Vue Router 4.x** - Client-side routing
- **Axios** - HTTP client

### External Services
- **DeepSeek API** - LLM for debate generation
- **SiliconFlow API** - Primary TTS service (CosyVoice2-0.5B)
- **Huawei Cloud** - Backup TTS service
- **Volcano Engine** - Image generation for posters

## Project Structure

```
.
├── main.py                 # CLI entry point (interactive debate)
├── server.py               # FastAPI server entry point
├── requirements.txt        # Base Python dependencies
├── requirements_api.txt    # API dependencies (+FastAPI, Uvicorn, websockets)
├── .env.sample            # Environment template (minimal)
├── .env.example           # Environment template (full)
├── Dockerfile             # Container configuration
├── debates.jsonl          # Debate data storage (JSONL format)
│
├── api/                   # FastAPI/WebSocket modules
│   ├── __init__.py
│   ├── agents.py          # AutoGen agent creation (pro/con/judge)
│   ├── config.py          # Environment configuration
│   ├── debate.py          # Debate orchestration logic
│   ├── debate_view.py     # Debate view routes
│   ├── models.py          # Pydantic data models
│   ├── routes.py          # FastAPI route definitions
│   ├── speech.py          # TTS service implementation
│   └── websocket.py       # WebSocket connection manager
│
├── flask/                 # Flask REST API service
│   ├── app.py             # Flask app factory
│   ├── api.md             # API documentation
│   ├── debates.py         # Debate CRUD API
│   ├── debatefromnews.py  # News-to-debate generation
│   ├── photo.py           # Image generation API
│   ├── ttv.py             # SiliconFlow TTS
│   ├── ttv2.py            # Alternative TTS
│   ├── ttv3.py            # Huawei Cloud TTS
│   └── debates.jsonl      # Local copy of debate data
│
├── frontend/              # Vue.js frontend
│   ├── package.json       # Node dependencies
│   ├── vite.config.js     # Vite configuration
│   ├── index.html         # Entry HTML
│   └── src/
│       ├── main.js        # App entry
│       ├── App.vue        # Root component
│       ├── router/        # Vue Router config
│       ├── config/        # API config
│       ├── assets/        # Static assets
│       ├── components/    # Reusable components
│       └── views/         # Page components
│           ├── Login.vue
│           ├── HotDebates.vue
│           ├── DebateView.vue
│           ├── AddDebateTopic.vue
│           ├── Discover.vue
│           └── UserProfile.vue
│
├── model/                 # Model training (optional)
│   ├── train.py
│   ├── data.py
│   ├── test_app.py
│   └── customize_service.py
│
├── tests/                 # Test suite
│   ├── test_debate_api.py
│   ├── test_websocket.py
│   ├── test_llm.py
│   ├── test_speech.py
│   └── run_tests.py
│
├── audio_output/          # Generated audio files (Flask)
└── speech_output/         # Generated audio files (FastAPI)
```

## Configuration

### Environment Variables

Create `.env` file from `.env.example`:

```bash
cp .env.example .env
```

**Required Variables:**
```env
# LLM Configuration (DeepSeek)
CUSTOM_LLM_API_KEY=sk-xxxxxxxx
CUSTOM_LLM_API_BASE=https://api.siliconflow.cn/v1
CUSTOM_LLM_MODEL=DeepSeek-V3

# Speech Configuration (SiliconFlow)
SPEECH_API_KEY=sk-xxxxxxxx
SPEECH_API_BASE=https://api.siliconflow.cn/v1
SPEECH_MODEL=FunAudioLLM/CosyVoice2-0.5B
SPEECH_VOICE=FunAudioLLM/CosyVoice2-0.5B:alex

# Server Settings
SERVER_HOST=0.0.0.0
SERVER_PORT=8001
CORS_ORIGINS=["*"]
```

**Optional Variables:**
```env
# Speech Feature Flags
SPEECH_ENABLED=true
SPEECH_CACHE_ENABLED=true
SPEECH_SAMPLE_RATE=32000
SPEECH_CHUNK_SIZE=100

# Per-agent Voice Config
SUPPORTER_VOICE=alloy
OPPONENT_VOICE=echo
JUDGE_VOICE=onyx
SUPPORTER_SPEED=1.0
OPPONENT_SPEED=1.0
JUDGE_SPEED=1.0

# Debate Settings
DEFAULT_MAX_ROUNDS=10
```

## Build and Run Commands

### Development Setup

```bash
# 1. Install Python dependencies
pip install -r requirements.txt

# 2. Configure environment
cp .env.example .env
# Edit .env with your API keys


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stephenlzc/AI-Agent-Debate_Autogen_Turtorial](https://github.com/stephenlzc/AI-Agent-Debate_Autogen_Turtorial) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
