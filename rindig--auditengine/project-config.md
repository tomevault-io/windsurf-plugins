---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ethics Engine is a web application for psychometric assessment of Large Language Models. It applies validated psychological instruments (authoritarianism scales, moral foundations questionnaires) to LLMs, measuring how models respond across different ideological framings ("personas").

The application supports two assessment modes:
1. **Text Assessment**: Traditional psychometric scales with Likert-type responses
2. **Visual Stimulus Assessment**: Image-based assessment using vision-capable models (e.g., Rorschach inkblots)

**Status**: Backend and frontend fully implemented with both text and visual assessment capabilities.

## Tech Stack

- **Frontend**: Next.js 14+ (App Router, TypeScript, shadcn/ui)
- **Backend**: FastAPI (Python, async-native)
- **Database**: Supabase (PostgreSQL with row-level security)
- **Job Queue**: Redis + Bull or Supabase Edge Functions
- **Hosting**: Vercel (frontend) + Railway/Fly.io (backend)

## Commands

### Backend (FastAPI)
```bash
cd backend
pip install -r requirements.txt
uvicorn app.main:app --reload    # Dev server
pytest                           # Run tests
black app/                       # Format code
flake8 app/                      # Lint
```

### Frontend (Next.js)
```bash
cd frontend
npm install
npm run dev                      # Dev server (port 3000)
npm run build                    # Production build
npm test                         # Run tests
npm run lint                     # ESLint
```

## Architecture

```
Frontend (Next.js) ──HTTP/WebSocket──▶ Backend (FastAPI) ──SQL──▶ Supabase
                                              │
                                              ├──▶ OpenAI API (text + vision)
                                              ├──▶ Anthropic API (text + vision)
                                              ├──▶ Google Gemini API (text + vision)
                                              ├──▶ xAI/Grok API
                                              ├──▶ DeepSeek API
                                              ├──▶ Groq API (text + vision)
                                              └──▶ Llama endpoints
```

Key data flow (Text Assessment):
1. User configures API keys, selects scales/models/personas in frontend
2. Backend orchestrates async API calls with provider-specific rate limiting
3. Responses parsed using multi-strategy parser
4. Scores reverse-coded per scale definition
5. Results stored in Supabase, progress streamed via WebSocket

Key data flow (Visual Stimulus Assessment):
1. User uploads image, selects vision-capable models and personas
2. Image sent as base64 with prompt to each model/persona combination
3. Open-ended text responses collected (no parsing/scoring)
4. Results available for download as CSV/JSON

## API Key Security & Flow

**Important**: User API keys are NEVER stored on the server. They flow through the system in-memory only.

### Production Flow (Vercel → Railway)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ 1. BROWSER (Vercel)                                                         │
│    User enters API keys → Stored in React state (browser memory only)       │
│    Keys validated via POST /api/keys/validate                               │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼ HTTPS POST /api/jobs
                                      │ (keys in request body)
┌─────────────────────────────────────────────────────────────────────────────┐
│ 2. BACKEND (Railway)                                                        │
│    jobs.py: create_job() receives keys in CreateJobRequest                  │
│    Keys passed to background task run_job_async()                           │
│    orchestrator.py: Keys used to create provider instances (in-memory)      │
│    LLM calls made → Keys discarded when job completes                       │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Key Storage Summary

| Location | Storage Type | Duration |
|----------|--------------|----------|
| Browser (Vercel) | React state | Until page refresh/close |
| Network | HTTPS request body | Transit only |
| Railway backend | In-memory (function params) | Duration of job only |
| Database | **Never stored** | N/A |

### Key Files in the Flow

1. **Frontend input**: `frontend/src/components/wizard/APIKeysSection.tsx`
   - User enters keys, stored in React state
   - Validation calls `validateAPIKey()` from `api.ts`

2. **Frontend submission**: `frontend/src/app/page.tsx` (handleStartAssessment)
   - Builds `apiKeyConfigs` array from validated keys
   - Calls `createJob()` with keys in request body

3. **Backend validation**: `backend/app/routers/keys.py`
   - POST `/api/keys/validate` - tests key validity with provider

4. **Backend job creation**: `backend/app/routers/jobs.py`
   - POST `/api/jobs` - receives keys in `CreateJobRequest.api_keys`
   - `run_job_async()` creates providers with keys, runs assessment


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RinDig/AuditEngine](https://github.com/RinDig/AuditEngine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
