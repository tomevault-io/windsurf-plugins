---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**LLMHub** is a full-stack AI collaboration platform with computer automation capabilities. It features a Next.js frontend with a FastAPI Python backend that orchestrates multi-agent AI systems capable of browser automation, terminal operations, and desktop control through containerized virtual machines.

## Architecture

### Frontend (Next.js 15 + React 19)
- **Framework**: Next.js 15 with App Router, TypeScript, Tailwind CSS
- **State Management**: Zustand stores for chat, models, user, and sessions
- **Key Libraries**:
  - Vercel AI SDK (`ai`) for streaming LLM responses
  - Radix UI for accessible components
  - Supabase for authentication and database
  - Stripe for billing/subscriptions
- **Provider System**: Multi-provider AI support (OpenAI, Anthropic, Azure, Google, Mistral, xAI, OpenRouter, Perplexity)

### Backend (Python FastAPI)
- **Framework**: FastAPI with async/await patterns
- **Key Services**:
  - `multi_agent_executor.py`: Orchestrates multi-agent task execution with browser, terminal, and desktop agents
  - `vm_control.py`: WebSocket-based VM control with persistent connections and auto-reconnection
  - `database.py`: Supabase integration for user data, chats, and billing
  - `agent_billing.py`: Tracks usage and credits for agent sessions
  - `search.py`: Google Custom Search API integration
- **API Routes**: `/api/chat`, `/api/models`, `/api/search`, `/api/vm`, `/api/billing`, `/api/files`

### VM Agent System
- **Architecture**: Docker containers running Ubuntu 22.04 with XFCE desktop
- **Agent Types**:
  - **Browser Agent**: Web automation using Chrome with remote debugging (search-first strategy)
  - **Terminal Agent**: Command execution and file operations
  - **Desktop Agent**: UI automation with screenshot analysis
- **Communication**: WebSocket protocol on port 8080 (8081 for localhost)
- **Tools**: Each agent has specialized tools (browser navigation, terminal commands, desktop controls)

### Key Design Patterns

#### Multi-Agent Execution Flow
1. **Task Planning**: LLM decomposes user request into sequential subtasks
2. **Agent Assignment**: Each subtask assigned to specialized agent (browser/terminal/desktop)
3. **Sequential Execution**: Tasks execute in order (no dependencies system)
4. **Context Passing**: Previous task summaries passed to next task for context
5. **Streaming**: All execution streams via Server-Sent Events to frontend

#### Provider Architecture
- Located in `lib/providers/` and `backend/app/providers/`
- Each provider implements streaming chat with tool calling
- Frontend providers handle model selection and API routing
- Backend providers execute tools and manage agent workflows

#### State Management
- **Chat Store** (`lib/chat-store/`): Manages conversations, messages, attachments
- **Model Store** (`lib/model-store/`): Available models and provider configurations
- **User Store** (`lib/user-store/`): User profile and authentication state
- **VM Store** (`lib/vm-store/`): Virtual machine sessions and connections

## Development Commands

### Frontend Development

```bash
# Install dependencies
npm install

# Development server (with Turbopack)
npm run dev

# Production build
npm run build

# Start production server
npm start

# Type checking
npm run type-check

# Linting
npm run lint
```

### Backend Development

```bash
# Navigate to backend directory
cd backend

# Create virtual environment (first time)
python -m venv venv

# Activate virtual environment
# Windows:
venv\Scripts\activate
# Linux/Mac:
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Run development server (from backend directory)
python main.py

# Or use the helper script
# Windows:
.\run_backend.bat
# Linux/Mac:
./run_backend.sh
```

### Docker Deployment

```bash
# Build and start all services
docker-compose up --build

# Start services in detached mode
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs -f

# AI Desktop container (separate compose file)
docker-compose -f docker-compose.ai-desktop.yml up --build
```

### Testing

```bash
# Backend tests
cd backend
pytest

# Run specific test file
pytest tests/test_specific.py

# Run with coverage
pytest --cov=app tests/
```

## Environment Configuration

### Frontend Environment Variables (.env)
- `NEXT_PUBLIC_SUPABASE_URL`: Supabase project URL
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`: Supabase anonymous key
- `SUPABASE_SERVICE_ROLE`: Supabase service role key (server-side)
- `CSRF_SECRET`: CSRF protection secret (required)
- `ENCRYPTION_KEY`: For encrypting user API keys (required for BYOK)
- `PYTHON_BACKEND_URL`: Backend API URL (default: http://0.0.0.0:8001)
- `NEXT_PUBLIC_BACKEND_URL`: Public backend URL (default: http://localhost:8001)
- Azure credentials for VM provisioning (AZURE_*)
- Stripe keys for billing (STRIPE_*)
- Google Search API keys (GOOGLE_SEARCH_*)

### Backend Environment Variables (backend/.env)
- `DEBUG`: Enable debug mode (true/false)
- `CORS_ORIGINS`: Allowed CORS origins (comma-separated)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vegedon/open-computer-use](https://github.com/vegedon/open-computer-use) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
