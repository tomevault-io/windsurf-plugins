---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an AI-powered meeting assistant that uses NVIDIA Nemotron LLM and LangGraph to orchestrate multi-agent workflows. The system analyzes meeting transcripts, manages calendar events, finds related meetings, and automatically sends HTML-formatted email summaries with timezone support.

**Frontend**: React 19 + TypeScript + Material-UI (MUI) on port 3000
**Backend**: Flask + LangGraph + NVIDIA Nemotron on port 5000
**Architecture**: Separate frontend/backend with Server-Sent Events (SSE) streaming for real-time workflow visualization

## Project Structure

```
.
├── frontend/                           # React 19 + TypeScript + MUI application
│   ├── src/
│   │   ├── components/                 # React components (Recording, Workflow, Results)
│   │   ├── hooks/                      # Custom hooks (useMediaRecorder, useWorkflowStream, useOrchestrator)
│   │   ├── context/                    # Global state (WorkflowContext)
│   │   ├── types/                      # TypeScript interfaces (workflow.ts)
│   │   └── theme/                      # NVIDIA green theme with MUI overrides
│   ├── package.json                    # Frontend dependencies + proxy config
│   └── public/index.html                # HTML entry point
├── python-clients/                     # NVIDIA Riva/NIM SDK package
│   ├── riva/                           # Riva gRPC client wrappers
│   ├── scripts/asr/                    # ASR utilities (used by server.py)
│   ├── tests/                          # Unit and integration tests
│   └── setup.py                        # Package metadata
├── Dockerfile.backend                  # Multi-stage Docker build for Flask backend
├── Dockerfile.frontend                 # Multi-stage Docker build for React + nginx
├── server.py                           # Flask API server (transcribe, orchestrate, stream-workflow)
├── orchestrator_agent.py               # LangGraph workflow orchestration (9 agents)
├── calender_tool.py                    # Google Calendar API integration
├── email_tool.py                       # Gmail API for sending summaries
├── attendee_mapping.json               # Name-to-email mappings for calendar
├── requirements.txt                    # Full Anaconda export (all dependencies)
├── requirements-docker.txt             # Minimal dependencies for Docker
└── CLAUDE.md                           # This file
```

## Development Commands

### Backend Setup & Running

```bash
# Install dependencies
pip install -r requirements.txt

# Set NVIDIA API key
export API_KEY="your_nvidia_api_key_here"

# Start Flask server (handles /transcribe endpoint + SSE streaming)
python server.py  # Listens on http://localhost:5000

# Run main orchestrator workflow with test transcript (standalone)
python orchestrator_agent.py

# Test Google Calendar integration (requires OAuth setup)
python calender_tool.py
```

### Frontend Setup & Running

```bash
cd frontend

# Install dependencies
npm install

# Start development server (port 3000, proxies API calls to localhost:5000)
npm start

# Run Jest tests in watch mode
npm test

# Build for production
npm run build

# Lint TypeScript/React code
npm run lint
```

### Docker Build & Deployment

```bash
# Build backend Docker image (multi-stage build)
docker build -f Dockerfile.backend -t nemotron-backend:latest .

# Build frontend Docker image (multi-stage with nginx)
docker build -f Dockerfile.frontend -t nemotron-frontend:latest .

# Run backend container
docker run -p 5000:5000 \
  -e API_KEY="your_nvidia_api_key" \
  -v $(pwd)/uploads:/app/uploads \
  nemotron-backend:latest

# Run frontend container
docker run -p 3000:80 \
  --link nemotron-backend:nemotron-backend \
  nemotron-frontend:latest
```

**Note**: Dockerfiles use `requirements-docker.txt` (minimal dependencies) instead of full `requirements.txt` (Anaconda export)

### Testing

**Backend Tests** (python-clients/tests):
```bash
# Run unit tests
cd python-clients
python -m pytest tests/unit/

# Run with verbose output
python -m pytest tests/unit/ -v

# Run specific test file
python -m pytest tests/unit/test_nlp.py

# Run integration tests (requires NVIDIA API credentials)
python -m pytest tests/integration/ -v
```

**Full Stack Testing**:
```bash
# Terminal 1: Start backend
python server.py

# Terminal 2: Start frontend
cd frontend && npm start

# Terminal 3: Test workflow
# Open http://localhost:3000 and record audio to test end-to-end
```

## Port Configuration

**Development Mode**:
- Frontend: http://localhost:3000 (React dev server)
- Backend: http://localhost:5000 (Flask server)
- Frontend package.json proxies API requests to http://localhost:4000 (configurable)

**Docker Mode**:
- Frontend: http://localhost:3000 (nginx serving built React SPA)
- Backend: http://localhost:5000 (Flask inside container)
- Nginx proxies `/api`, `/transcribe`, `/stream-workflow`, `/orchestrate` requests to backend service

## Environment Configuration

### Setup Required Files

1. **`.env` file** (or export variables):
   ```bash
   export API_KEY="your_nvidia_api_key"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [biraj-dahal/Nvidia-Nematron-Agentic-Workflow](https://github.com/biraj-dahal/Nvidia-Nematron-Agentic-Workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-25 -->
