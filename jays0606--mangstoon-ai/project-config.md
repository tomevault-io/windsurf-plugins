---
trigger: always_on
description: AI 망상툰 generator. **3rd Place — Gemini 3 Seoul Hackathon, Feb 28 2026 — $20,000 Credits**
---

# MangstoonAI — CLAUDE.md

AI 망상툰 generator. **3rd Place — Gemini 3 Seoul Hackathon, Feb 28 2026 — $20,000 Credits**

---

## Project Structure

```
gemini-hackathon/
├── CLAUDE.md
├── README.md
├── .gitignore                        # .env, output/, node_modules/ all ignored
├── docs/
│   ├── hackathon-info.md             # Schedule, rules, judging, prizes
│   ├── gemini-3.md                   # Gemini 3.1 Pro/Flash API reference
│   ├── nano-banana-image-generation.md  # Image gen API + prompting
│   ├── adk-llms.txt                  # ADK Python SDK reference
│   └── prompt-design.md              # Gemini prompting best practices
├── cloudbuild.yaml                   # Cloud Build config (alternative to GitHub Actions)
├── .github/workflows/
│   └── deploy-backend.yml            # CI/CD: push to main → Cloud Run deploy
├── backend/                          # FastAPI + ADK agent (Python, uv)
│   ├── .env                          # GOOGLE_API_KEY — gitignored, never commit
│   ├── Dockerfile                    # Python 3.12-slim + pip install
│   ├── .dockerignore                 # Excludes .env, output/, __pycache__/
│   ├── main.py                       # FastAPI app (POST /generate, POST /edit, GET /health)
│   ├── pyproject.toml                # uv managed dependencies
│   ├── requirements.txt              # pip deps for Docker (mirrors pyproject.toml)
│   ├── run.sh                        # ./run.sh [--reload] — starts FastAPI on :8000
│   ├── output/panels/                # Generated PNG files — gitignored
│   └── mangstoon_ai/                 # ADK agent package
│       ├── __init__.py
│       ├── agent.py                  # root_agent — LlmAgent with 3 tools + system prompt
│       ├── gcs.py                    # upload_panel() → GCS public URL
│       ├── styles.py                 # 4 art style definitions (prompts, configs, names)
│       ├── tools/
│       │   ├── story_engine.py       # decompose_story() — Flash → 22-panel JSON storyboard
│       │   ├── image_gen.py          # generate_panel() — 2-step: Flash optimize → Flash Image + GCS
│       │   ├── panel_editor.py       # edit_panel() — 2-step: Flash edit prompt → Flash Image + GCS
│       │   └── character.py          # extract_character() — selfie → character description
│       └── prompts/
│           └── system.py             # MANGSTOON_DIRECTOR_INSTRUCTION — 3-phase director
└── frontend/                         # Next.js 14 app (deploy to Vercel)
    ├── app/
    │   ├── layout.tsx                # Black Han Sans + Noto Sans KR, title metadata
    │   ├── globals.css               # CSS variables, animations, split layout classes
    │   ├── page.tsx                  # Root: Phase state machine (0=style, 1=input, 2=viewer)
    │   └── components/
    │       ├── ChatPanel.tsx         # Left panel: StoryInput + ChatEditor
    │       ├── StoryInput.tsx        # Story textarea + selfie upload
    │       ├── WebtoonViewer.tsx     # Right panel: 22 panels, speech bubbles, loading skeletons
    │       └── ChatEditor.tsx        # KakaoTalk-style edit chat
    └── app/api/
        ├── generate/route.ts         # Proxy → backend:8000/generate
        └── edit/route.ts             # Proxy → backend:8000/edit
```

---

## API Keys & Environment

```bash
# backend/.env  (gitignored — never commit)
GOOGLE_API_KEY=your_key_here
```

The server loads `backend/.env` on startup via `load_dotenv(Path(__file__).parent / ".env")`.
Frontend has no secrets — it proxies to the backend via Next.js API routes.

### Production (Cloud Run)
`GOOGLE_API_KEY` is stored in **Secret Manager** and mounted as env var via `--set-secrets`.
GCS auth uses the Cloud Run service account (`mangstoon-deployer@...`) — no extra config needed.

---

## Models

| Step | Model | Thinking | Purpose |
|------|-------|----------|---------|
| Orchestrator | `gemini-3.1-pro-preview` | low | Directs 3-phase flow, calls tools, chats |
| `decompose_story` | `gemini-3-flash-preview` | low | 22-panel JSON storyboard from user story |
| `generate_panel` step 1 | `gemini-3-flash-preview` | minimal | Optimizes image prompt from panel metadata |
| `generate_panel` step 2 | `gemini-3.1-flash-image-preview` | — | Generates 9:16 panel image |
| `edit_panel` step 1 | `gemini-3-flash-preview` | minimal | Creates updated prompt with edit applied |
| `edit_panel` step 2 | `gemini-3.1-flash-image-preview` | — | Regenerates edited panel |
| `extract_character` | `gemini-3.1-pro-preview` | low | Selfie → character description |

**Do NOT use** `gemini-3-pro-preview` — deprecated, shuts down March 9.

---

## Running Locally

```bash
# ADK dev UI — run from backend/ (mandatory hackathon demo)
cd backend
uv run --project .. adk web --port 8080
# → http://localhost:8080/dev-ui/  →  select mangstoon_ai

# FastAPI backend
cd backend
./run.sh              # port 8000
./run.sh --reload     # with hot reload

# Next.js frontend
cd frontend
npm run dev           # port 3000
```

---

## Agent Architecture (3-Phase Pipeline)

### ADK path (adk web — hackathon demo)
```
User message
  → root_agent (gemini-3.1-pro-preview + MANGSTOON_DIRECTOR_INSTRUCTION)
      Phase 1: calls decompose_story() → presents storyboard → asks for approval

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jays0606/mangstoon_ai](https://github.com/jays0606/mangstoon_ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
