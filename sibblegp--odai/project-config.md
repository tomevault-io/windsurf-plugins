---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Commands

### Testing
```bash
# Run all tests with advanced test runner
python run_tests.py                               # Run all tests with parallel execution
python run_tests.py --coverage                    # Run tests with coverage report
python run_tests.py --file auth_service           # Run specific test file
python run_tests.py --verbose                     # Run tests with verbose output
python run_tests.py --workers 8                   # Run tests with 8 parallel workers

# Direct pytest commands
pytest tests/                                     # Run all tests
pytest tests/test_auth_service.py                 # Run specific test file
pytest tests/ --cov=services --cov-report=html    # Generate HTML coverage report
pytest tests/ -xvs                                # Stop on first failure with verbose output
```

### Development
```bash
# Install dependencies
pip install -r requirements.txt                   # 173 production dependencies
pip install -r test_requirements.txt              # Testing dependencies

# Run locally (set LOCAL=true in environment)
uvicorn api:APP --reload                          # Run with auto-reload
uvicorn api:APP --host 0.0.0.0 --port 8080       # Run on specific host/port

# Deploy to environments
./deploy_development.sh                           # Deploy to dev (odai-dev-5e4fd)
./deploy_production.sh                            # Deploy to prod (odai-prod)
```

## Project Overview

ODAI is a sophisticated AI assistant platform built with FastAPI, featuring:
- **30+ Third-Party Integrations**: Financial, travel, communication, entertainment services
- **Real-time WebSocket Support**: Streaming chat responses with connection lifecycle management
- **Voice Interface**: Twilio integration for voice-based AI interactions
- **Enterprise Security**: Google Cloud KMS encryption, OAuth 2.0 authentication
- **Comprehensive Testing**: 700+ tests with 90%+ coverage across 67 test files
- **Cloud-Native Architecture**: Google App Engine with auto-scaling (2-20 instances in production)

## Code Architecture

### High-Level Structure
```
api.py (FastAPI main)
├── routers/              # API endpoints & route handlers
│   ├── app_voice.py      # Voice interaction endpoints
│   ├── google.py         # Google services integration
│   ├── twilio_handler.py # SMS/voice handling
│   └── voice_utils/      # Audio processing utilities
├── services/             # Business logic layer
│   ├── auth_service.py   # Authentication & OAuth
│   ├── chat_service.py   # OpenAI chat management
│   └── api_service.py    # Non-chat API endpoints
├── connectors/           # 30+ third-party integrations
│   ├── orchestrator.py   # Agent tool orchestration
│   └── [integration].py  # Individual service connectors
├── firebase/models/      # Data models & encryption
│   ├── user.py           # User profiles
│   ├── chat.py           # Chat sessions
│   └── [model]_token.py  # Encrypted token storage
├── websocket/            # Real-time communication
│   └── handlers/         # WebSocket connection handlers
├── middleware/           # Application middleware
└── tests/                # 67 test files, 700+ tests
```

### Core Components

#### 1. **API Layer** (`api.py`, `routers/`)
- FastAPI application with CORS middleware and authentication
- RESTful endpoints + WebSocket support for real-time chat
- Routers:
  - `app_voice.py`: Voice-based AI interactions
  - `google.py`: Google OAuth and services (Calendar, Docs, Gmail)
  - `plaid.py`: Banking integration
  - `evernote.py`: Note-taking integration
  - `twilio_handler.py`: SMS/voice communication

#### 2. **Service Layer** (`services/`)
- `auth_service.py`: Google OAuth, JWT tokens, user management (84% coverage)
- `chat_service.py`: OpenAI integration, message processing, agent tools (97% coverage)
- `api_service.py`: Non-chat API endpoint orchestration
- `location_service.py`: IP-based geolocation services

#### 3. **WebSocket Layer** (`websocket/`)
- Real-time bidirectional communication for streaming chat
- Connection lifecycle management with authentication
- Message queuing and error handling
- Support for both text and voice interactions

#### 4. **Data Layer** (`firebase/models/`)
- **User Management**: `user.py` - profiles, preferences, integration settings
- **Chat Storage**: `chat.py` - sessions, messages, conversation history
- **Token Management**: Encrypted storage using Google Cloud KMS
  - `google_token.py`: Google OAuth tokens
  - `plaid_token.py`: Banking credentials
  - `evernote_token.py`: Note service tokens
- **Analytics**: `token_usage.py` - OpenAI API usage and cost tracking
- **Error Tracking**: `unhandled_request.py` - Failed request logging

#### 5. **Integration Layer** (`connectors/`)

##### Financial & Banking
- `plaid_agent.py`: Bank account access and transactions
- `finnhub_agent.py`: Stock market data and financial news
- `coinmarketcap.py`: Cryptocurrency prices and market data
- `exchange_rate.py`: Currency conversion rates

##### Travel & Transportation
- `amadeus_agent.py`: Flight booking and travel planning

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sibblegp/ODAI](https://github.com/sibblegp/ODAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
