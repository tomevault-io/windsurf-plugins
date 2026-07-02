---
trigger: always_on
description: **intervyu.io** is an AI-powered interview preparation platform. Candidates do real voice interviews with an AI interviewer (Neerja), get their code evaluated live, upload their CV, and receive a performance report at the end.
---

# intervyu - Claude Code Context

## What This Project Is
**intervyu.io** is an AI-powered interview preparation platform. Candidates do real voice interviews with an AI interviewer (Neerja), get their code evaluated live, upload their CV, and receive a performance report at the end.

## Project Structure

```
intervyu/
├── frontend/                    # Next.js 15 (React 19, TypeScript, Tailwind CSS 4)
│   ├── app/
│   │   ├── page.tsx             # Home — interview type selection + dashboard
│   │   ├── layout.tsx           # Root layout (Geist fonts)
│   │   ├── login/               # OAuth login page (Google + GitHub)
│   │   ├── auth/callback/       # Supabase OAuth callback handler
│   │   ├── interview/new/       # Live interview session page
│   │   └── demo/                # Demo pages (code-editor, cv, performance)
│   ├── components/
│   │   ├── VoiceInterview.tsx   # Main voice interview + WebSocket logic
│   │   ├── Sidebar.tsx          # Nav sidebar with user avatar + sign-out
│   │   ├── home/
│   │   │   ├── InterviewCard.tsx
│   │   │   ├── StartInterviewModal.tsx
│   │   │   ├── PastInterviewsList.tsx
│   │   │   ├── StatsCard.tsx
│   │   │   └── ScheduleModal.tsx
│   │   ├── code-editor/CodeEditor.tsx
│   │   ├── cv/CVUpload.tsx + CVAnalysisDisplay.tsx
│   │   ├── performance/PerformanceDashboard.tsx + InterviewHistory.tsx
│   │   └── common/PDFExport.tsx
│   └── lib/
│       ├── supabase/client.ts   # Supabase singleton client
│       ├── supabase/auth.ts     # useSupabaseSession, useRequireAuth, OAuth helpers
│       └── api.ts               # Centralised authFetch + all API helpers + buildWsUrl
│
├── backend/                     # FastAPI (Python 3.11)
│   └── app/
│       ├── main.py              # App init, CORS, router registration, DB pool lifespan
│       ├── routers/
│       │   ├── sessions.py      # POST /api/sessions, GET /api/sessions/{id}
│       │   ├── websocket.py     # WS /ws/interview/{session_id}?token=<jwt>
│       │   ├── interviews.py    # CV upload, transcript, end session, report
│       │   ├── code.py          # Code execution via Lambda
│       │   ├── analytics.py     # Aggregate stats, benchmarks, trends (PostgreSQL)
│       │   └── auth.py          # GET /api/auth/me
│       ├── services/
│       │   ├── bedrock_service.py   # Claude Haiku 4.5 via bedrock-runtime
│       │   ├── db_service.py        # asyncpg pool + all CRUD (replaces S3 JSON)
│       │   ├── auth_service.py      # Supabase JWT verification (PyJWT HS256)
│       │   ├── s3_service.py        # Binary files only: CVs, audio
│       │   ├── lambda_service.py    # Lambda invocation helper
│       │   └── textract_service.py  # CV text extraction + skill parsing
│       ├── dependencies/
│       │   └── auth.py              # CurrentUser dataclass + get_current_user Depends
│       ├── models/
│       │   ├── session.py           # Session Pydantic models
│       │   └── code_submission.py   # Code metrics & test result models
│       └── config/
│           ├── settings.py          # Env-var settings (incl. Supabase + DATABASE_URL)
│           ├── interview_types.py   # 8 interview configs + phases
│           └── agent_instruction.txt # Neerja persona system prompt
│
├── lambda-tools/                # AWS SAM — 3 Lambda functions
│   ├── code-executor/           # Sandboxed Python/JS execution
│   ├── cv-analyzer/             # Resume parsing + skill categorization
│   ├── performance-evaluator/   # Score calculation + report generation
│   └── template.yaml            # SAM CloudFormation template
│
├── database/                    # Supabase PostgreSQL (active)
│   ├── supabase_schema.sql      # Live schema (run in Supabase SQL Editor)
│   ├── schema.sql               # Original local schema (reference only)
│   ├── docker-compose.yml       # Local Postgres dev (optional)
│   ├── migrations/
│   │   └── 002_s3_to_postgres_migration.sql  # insert_session_from_json() for backfill
│   └── scripts/migrate_from_s3.py  # S3 → Supabase migration script
│
├── knowledge-base/              # RAG content for Bedrock Agent
├── deployment/                  # AWS deployment guides
├── scripts/                     # deploy-frontend.sh, deploy-backend.sh
└── docs/                        # Architecture, optimization notes
```

## Tech Stack

### Backend (FastAPI)
- **STT**: Deepgram Nova-2 API (cloud, persistent `httpx.AsyncClient` module-level singleton to `api.deepgram.com/v1/listen`)
- **TTS**: Azure Cognitive Services Speech SDK (`azure-cognitiveservices-speech`, `en-IN-NeerjaNeural` voice, MP3 output `Audio24Khz48KBitRateMonoMp3`, pool of 3 persistent `SpeechSynthesizer` instances, SSML `<prosody rate="+20%">`, MP3 chunks streamed via asyncio.Queue concurrent sender)
- **AI**: Anthropic API direct (`anthropic` SDK, `claude-haiku-4-5-20251001`); conversation history managed manually in `_session_cache`. Previously used AWS Bedrock `converse_stream` — switched 2026-06 when new AWS account couldn't get Bedrock model access approved.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sahu-adarsh/intervyu](https://github.com/sahu-adarsh/intervyu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
