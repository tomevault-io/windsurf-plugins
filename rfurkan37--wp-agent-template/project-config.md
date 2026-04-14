---
trigger: always_on
description: **Name:** AI Chatbot Lite
---

# AI Chatbot Lite - Cursor Rules

## Project Overview

**Name:** AI Chatbot Lite
**Description:** Lightweight Gemini-powered customer support AI assistant with WhatsApp Cloud API integration. The bot maintains conversation history through Gemini's native chat sessions and uses a Markdown knowledge base for grounded responses—no vector databases or heavy RAG stack required.

**Stack:**
- Language: Python 3.11+
- AI: Google Gemini SDK
- Web Framework: FastAPI + Uvicorn
- WhatsApp Integration: PyWa library
- Package Manager: uv
- Deployment: Google Cloud Run (Docker containers)

**Key Features:**
- Multi-language support (12+ languages)
- Markdown-based knowledge base (no vector DB needed)
- Optional WhatsApp integration (gracefully disabled if credentials missing)
- Session-based conversation history
- Health monitoring endpoints

---

## ⚠️ CRITICAL AI ASSISTANT RULES ⚠️

### NEVER CREATE DOCUMENTATION FILES

**ABSOLUTELY FORBIDDEN:**
- ❌ NEVER create .md files (README.md, GUIDE.md, DEPLOYMENT.md, etc.)
- ❌ NEVER create documentation files in any format unless EXPLICITLY requested by user
- ❌ NEVER create summary files, guide files, or reference files
- ❌ NEVER suggest creating documentation files

**INSTEAD:**
- ✅ Provide all information directly in the chat response
- ✅ Answer questions thoroughly inline
- ✅ Use code blocks and formatting in chat
- ✅ Keep all project documentation in THIS .cursorrules file only

**EXCEPTION:**
- Only create documentation files if the user explicitly says: "create a file named X.md" or similar direct request

---

### RELY ON CLOUD RUN AND DEPLOYMENT SERVICES (DON'T BUILD FROM SCRATCH)

**CRITICAL PRINCIPLE:**
- ❌ NEVER implement rate limiting, semaphores, concurrency controls, or load balancing from scratch
- ❌ NEVER build infrastructure features that Cloud Run already provides
- ❌ NEVER over-engineer solutions when the deployment platform handles it

**INSTEAD:**
- ✅ Use Cloud Run's built-in features:
  - Concurrency limits (configured in cloudbuild.yaml: `--concurrency`)
  - Autoscaling (min/max instances)
  - Request queuing and distribution
  - Load balancing (automatic)
  - Health checks (automatic)
- ✅ Configure these in `cloudbuild.yaml` or Cloud Run settings, NOT in application code
- ✅ Keep application code simple - handle business logic only
- ✅ Let Cloud Run handle infrastructure concerns

**REMEMBER:**
- We are users of deployment services, not building infrastructure
- Cloud Run is a managed service - use its features
- Keep code simple and focused on business logic

---

## System Architecture

### Component Structure
```
src/aiChatbot/
├── adapters/            # Channel adapters (WhatsApp via PyWa)
│   ├── pywaWhatsappAdapter.py   # PyWa-based WhatsApp adapter (active)
│   └── whatsappAdapter.py       # Legacy adapter (kept for rollback)
├── api/                 # FastAPI application
│   └── app.py          # Webhook endpoints + health checks
├── interfaces/         # Abstract base classes
│   ├── aiService.py
│   ├── channelAdapter.py
│   ├── embeddingService.py
│   └── messageProcessor.py
├── models/             # Data models
│   ├── botConfig.py    # Configuration loader
│   ├── chatSession.py  # Session state
│   └── standardMessage.py  # Unified message format
├── services/           # Core business logic
│   ├── channelManager.py        # Channel registration & routing
│   ├── geminiAIService.py       # Gemini SDK integration
│   ├── messageProcessorService.py  # Message handling pipeline
│   ├── serviceFactory.py        # Dependency injection
│   └── sessionManager.py        # Conversation state management
├── utils/              # Utilities
│   ├── languageDetector.py  # Automatic language detection
│   └── promptManager.py     # Centralized prompt management
├── main.py            # CLI entry point
└── asgi.py            # ASGI application for Cloud Run
```

### Message Flow
1. **Incoming Webhook** → FastAPI endpoint (`/webhook/whatsapp`)
2. **Channel Adapter** → Converts WhatsApp message to StandardMessage
3. **Message Processor** → Routes to AI service
4. **AI Service** → Gemini generates response with knowledge base context
5. **Channel Adapter** → Sends response back via WhatsApp Cloud API

### Configuration System
- **Environment Variables:** `.env` for local, Google Secret Manager for production
- **Prompts:** `data/prompts.json` (centralized, hot-reloadable)
- **Knowledge Base:** `data/knowledge-base.md` (simple Markdown file)
- **Bot Config:** `src/aiChatbot/models/botConfig.py` (loads from env)

---

## Cloud Run Deployment

### Required Secrets (Google Secret Manager)

Store these in Secret Manager (never commit to repo):
```
GEMINI_API_KEY              # Google AI Studio / Vertex API key
WHATSAPP_PHONE_NUMBER_ID    # WhatsApp Cloud API phone number ID (optional)
WHATSAPP_ACCESS_TOKEN       # WhatsApp permanent access token (optional)
WHATSAPP_WEBHOOK_VERIFY_TOKEN  # Token for webhook verification (optional)
```

### Deployment Commands

**Manual Deployment:**
```bash
# Deploy using script
./deploy-cloudrun.sh production

# Or deploy directly
gcloud run deploy your-service-name \
  --region europe-west1 \
  --source .
```

---

## Development Guidelines

### Setting Up Local Environment

```bash
# 1. Install uv package manager
pip install uv


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rfurkan37) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
