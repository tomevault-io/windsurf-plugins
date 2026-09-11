---
trigger: always_on
description: Orientation for AI coding assistants (Codex, Claude Code, etc.) working in this repository. For end-user docs, see `README.md`. For deeper technical detail, see `docs/architecture.md` and `docs/onboarding.md`.
---

# CLAUDE.md

Orientation for AI coding assistants (Codex, Claude Code, etc.) working in this repository. For end-user docs, see `README.md`. For deeper technical detail, see `docs/architecture.md` and `docs/onboarding.md`.

## What this project is

VEA is a video editing automation service. The current product is a **conversational editing agent** that runs in a React dashboard. A user drops video files into a workspace, the system indexes them into the hosted Memories.ai Video Datalake, and an LLM-driven agent collaborates with the user in chat to plan, refine, and compile a Final Cut Pro XML edit. Drafts auto-render via FFmpeg; high-quality finals can render via DaVinci Resolve.

The legacy V1 pipeline (videoComprehension → flexibleResponse → ...) and its Memories.ai cloud client are **not on this branch** — `src/pipelines/` holds only `common/` and `v2/`. The paper's original codebase lives on the `legacy/v1-main` branch; references to "V1" below are historical unless they name that branch.

## Key directories

```
src/
├── app.py                          # FastAPI entrypoint (port 8000)
├── services.py                     # Shared singletons (main_llm, video_llm, retrieval handles, agent sessions)
├── datalake.py                     # Video Datalake retrieval backend (VIDEO_BACKEND=datalake)
├── cli.py                          # One-shot CLI (vea-oneshot) for non-interactive runs
├── routes/                         # FastAPI routers
│   ├── _route_utils.py             # Path-safety helpers (workspace resolution)
│   ├── v2_pipelines.py             # REST: plan, FCPXML, narration, music, crop
│   ├── v2_projects.py              # REST: list/create/clear + /system/info + /system/model
│   └── v2_websockets.py            # WebSocket: agent chat + indexing progress
├── pipelines/
│   ├── v2/                         # ★ Current architecture
│   │   ├── agent/
│   │   │   ├── agent_session.py    # Agent loop, history, persistence
│   │   │   ├── tools.py            # Tool executor (functions invoked by LLM)
│   │   │   ├── tool_definitions.py # Gemini FunctionDeclarations (10 tools)
│   │   │   ├── tool_helpers.py     # ElevenLabs / ffmpeg helpers
│   │   │   ├── system_prompt.py    # System prompt template + builder
│   │   │   ├── scratchpad.py       # ScratchpadManager (4 persistent .md files)
│   │   │   └── timeline_view.py    # Programmatic timeline diagram for the prompt
│   │   ├── comprehension/
│   │   │   └── lightweight_comprehension.py   # Upload + gist (V2 indexing)
│   │   ├── fcpxml/
│   │   │   └── edit_compiler.py    # Deterministic EditDecision → FCPXML 1.10
│   │   ├── preview/
│   │   │   └── ffmpeg_renderer.py  # Auto draft renderer (no DaVinci needed)
│   │   ├── audio/
│   │   │   └── loudness.py         # ITU-R BS.1770 LUFS measurement (pyloudnorm)
│   │   ├── music/
│   │   │   └── beat_sync.py        # librosa beat detection (advisory, agent reads)
│   │   ├── workspace.py            # WorkspaceManager (file I/O for projects)
│   │   └── schemas.py              # SessionData, EditDecision, ClipDecision, etc.
│   ├── common/                     # Shared (V1+V2): TimelineConstructor, dynamic crop
├── schema.py                       # FastAPI request/response models
└── config.py                       # Config loading, paths, env var population
lib/
├── llm/
│   ├── GeminiGenaiManager.py       # Vertex AI Gemini client
│   └── OpenRouterManager.py        # OpenRouter (drop-in replacement for Gemini)
└── utils/
    ├── media.py                    # ffmpeg/ffprobe helpers
    ├── resolve_setup.py            # DaVinci Resolve health check / pythonpath
    └── resolve_render.py           # DaVinci Resolve render entrypoint
dashboard/                          # React + Vite + TypeScript frontend
└── src/
    ├── App.tsx
    ├── hooks/useAgentChat.ts       # WebSocket hook, manages all real-time state
    └── components/                 # AgentChat, NLETimeline, AudioInspector, ...
data/
└── workspaces/{project}/           # Per-project storage (footage, edits, renders)
scripts/
└── datalake_ingest.py              # upload footage to a datalake collection + write session.json
docs/                               # architecture.md, onboarding.md
tests/v2/                           # pytest suite (230+ tests, offline)
```

## Setup commands

```bash
# Install everything (Python deps via uv, dashboard deps + build)
uv sync
cd dashboard && npm install && npm run build && cd ..

# Configure
cp config.example.json config.json   # then fill in api_keys

# Start backend (also handles setup if missing)
./dev.sh up

# Or run by hand:
source .venv/bin/activate
python -m src.app
```

The dashboard is served at **http://localhost:8000/app**. API docs at **http://localhost:8000/docs**.

System dependency: `ffmpeg` must be installed (`brew install ffmpeg` or distro equivalent).

## LLM providers (main_llm vs video_llm)

V2 uses **two** LLM slots, both initialized in `src/services.py`:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Memories-ai-labs/vea-open-source](https://github.com/Memories-ai-labs/vea-open-source) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
