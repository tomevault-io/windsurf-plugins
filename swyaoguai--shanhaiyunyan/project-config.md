---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Common commands

### Setup and run
```bash
pip install -r requirements.txt
cp .env.example .env
python run.py
```

Notes:
- `run.py` is the real dev entrypoint. It loads `.env`, checks data-dir write permission, finds an available port starting from configured `PORT`/default `5656`, creates the FastAPI app, and starts Uvicorn.
- On startup the app may auto-switch to the next free port if `5656` is occupied.

### Backend tests
```bash
pytest
pytest novel_agent/tests
pytest novel_agent/knowledge_base/tests
```

Single test / filtered test:
```bash
pytest novel_agent/tests/test_short_story_service.py
pytest novel_agent/tests/test_short_story_service.py -k some_case_name
```

### Frontend tests
```bash
npm install
npm run test:frontend
npm run test:frontend:watch
```

Single frontend test file:
```bash
npx vitest run --pool threads --maxWorkers 1 frontend-tests/continuous-write.dom.test.js
```

### Release build
```bash
python build_release.py
```

Notes:
- The build scripts expect PyInstaller, and final installer EXE output requires Inno Setup (`ISCC.exe`).
- The release script builds exactly one Windows installer EXE file in `dist/`: the local-model version with `novel_agent/models/embedding/default` bundled. Do not publish the no-ONNX/lite installer.
- Release builds do not generate zip archives. `build_portable.py` is kept only for local portable-directory checks and also no longer creates zip files.
- The build scripts package `run.py`, bundle `novel_agent/web/static`, `novel_agent/web/templates`, `novel_agent/prompts`, Skills, and a clean release data copy.

### No lint command
- There is no dedicated root lint script in `package.json` or repo-level documented lint command. Do not invent one in changes or docs.

### Update log
- Record future version updates, bug fixes, UX changes, and test/build changes in `CHANGELOG.md`.
- Keep entries outcome-oriented: describe only what feature was added, what problem was fixed, what UX changed, or what test/build result changed.
- Do not explain implementation mechanisms, investigation process, internal call chains, or low-level vendor details in `CHANGELOG.md`; include technical names only when needed to identify the affected feature or problem.
- Keep entries concise. The current product version line starts at `v1.0.0`; during future development, add pending items under `[未发布]` only when a release section has not been chosen yet.

## High-level architecture

### Runtime startup flow
- `run.py` is the outer runtime wrapper.
- `run.py` calls `novel_agent.web.create_app()` and runs Uvicorn.
- `novel_agent/web/app.py` is the FastAPI application factory and lifecycle glue.
- In app lifespan startup, the app:
  - validates startup config
  - initializes global config
  - starts cache cleanup
  - creates `NovelCoordinator`
  - creates `RouterAgent`
  - optionally wires router knowledge-base access
  - starts the message bus
  - installs a default message handler so unhandled requests fall back to `RouterAgent`

Important files:
- `run.py`
- `novel_agent/web/app.py`
- `novel_agent/web/dependencies.py`
- `novel_agent/web/routes/__init__.py`

### Backend composition
- The backend is a FastAPI app with modular routers under `novel_agent/web/routes/`.
- Route registration happens in `novel_agent/web/routes/__init__.py`.
- APIs are mounted twice:
  - `/api/v1/*` for versioned endpoints
  - `/api/*` for backward compatibility
- `novel_agent/web/app.py` also configures CORS, rate limiting, static files, templates, and log sanitization.

### Coordinator-worker writing architecture
- The core writing engine is `NovelCoordinator` in `novel_agent/workflow/coordinator.py`.
- It owns workflow state, checkpoints, project switching, and orchestration across specialist agents.
- It instantiates and coordinates:
  - `WorldbuilderAgent`
  - `OutlinerAgent`
  - `ChapterWriterAgent`
  - `PolisherAgent`
  - `EvaluatorAgent`
- It also wires in:
  - `ContextManager`
  - `CharacterManager`
  - `WorldManager`
  - memory/aux-memory services
  - message bus and metrics

This is the main “backend brain” for novel generation and project-scoped writing state.

### Router-first request handling
- `novel_agent/agents/router_agent.py` is the first-level intent router for user requests.
- It uses keyword/entity analysis to classify requests into creation, continuation, polishing, web search, trends, knowledge queries, project/config operations, or general chat.
- Depending on intent it routes to:
  - `NovelCoordinator`
  - continuous writing
  - polishing flow
  - communicator/chat flow
  - Skill calls for web/trend operations
- The router can consult the knowledge base before responding.

### Base agent / LLM configuration model
- `novel_agent/agents/base_agent.py` centralizes shared agent behavior.
- Agents use `openai.AsyncOpenAI` against OpenAI-compatible endpoints.
- Timeouts come from timeout settings, and retries are handled by project logic rather than SDK retries.
- `novel_agent/agent_config.py` defines the effective model config system:
  - multiple saved API configs
  - active config/model selection
  - per-agent overrides

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swyaoguai/shanhaiyunyan](https://github.com/swyaoguai/shanhaiyunyan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
