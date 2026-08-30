---
trigger: always_on
description: Open Executive is a multi-agent AI system acting as a virtual corporate executive. Python backend (FastAPI) + Next.js 15 frontend. The "Executive" is a single coherent persona backed by 8 specialist sub-agents, all powered by the Anthropic Claude API.
---

# Open Executive — Claude Code Context

## Project Overview

Open Executive is a multi-agent AI system acting as a virtual corporate executive. Python backend (FastAPI) + Next.js 15 frontend. The "Executive" is a single coherent persona backed by 8 specialist sub-agents, all powered by the Anthropic Claude API.

## Repository Layout

```
packages/core/          Python: all agent logic, API, CLI
packages/ui/            Next.js 15 web UI
knowledge/              Curated MBA knowledge base (git-tracked Markdown)
evals/                  Eval scenarios + LLM-as-judge runner
docker/                 Dockerfile + docker-compose.yml
.github/workflows/      CI + eval pipeline
```

## Key Commands

```bash
make dev          # Start FastAPI (port 8000) + Next.js (port 3000)
make test         # Run pytest
make lint         # ruff check + mypy
make eval         # Run eval suite against localhost
make docker       # docker compose up --build
```

## Python Setup

Uses `uv` for package management.

```bash
cd packages/core
uv sync
source .venv/bin/activate
```

## Architecture — How the Agent System Works

1. User message arrives at `Executive` (orchestrator in `orchestrator/executive.py`)
2. Executive uses Anthropic tool use to call `consult_specialist` for relevant domains
3. For cross-domain questions, multiple specialists are called in parallel
4. Each specialist:
   - Gets its domain system prompt from `prompts/domain_prompts.py`
   - Retrieves relevant chunks from ChromaDB (built-in knowledge + company docs)
   - Returns analysis to the Executive
5. Executive synthesizes all specialist input into one coherent response
6. The internal agent architecture is NEVER exposed to the user

## Prompt Caching — Critical

The system is designed around Anthropic prompt caching. Breaking caching = 10x cost increase.

**Never put dynamic content in system prompt blocks that have `cache_control`.**

Build order in `prompts/cache_manager.py`:
1. Tool definitions (sorted by name — MUST be sorted)
2. Executive persona constant (from `prompts/executive_persona.py` — NEVER f-stringed)
3. Company profile block (from `memory/company_profile.py`)
4. Knowledge index summary

RAG context goes in the **user turn**, not the system prompt.

## Adding a New Specialist Agent

1. Create `packages/core/openexecutive/agents/your_agent.py`:
   ```python
   from openexecutive.agents.base import BaseAgent
   
   class YourAgent(BaseAgent):
       name = "your_agent"
       domain = "your_domain"
       model = "claude-sonnet-4-6"
       
       def get_system_prompt(self) -> str:
           from openexecutive.prompts.domain_prompts import YOUR_AGENT_PROMPT
           return YOUR_AGENT_PROMPT
   ```

2. Add `YOUR_AGENT_PROMPT` constant to `prompts/domain_prompts.py`

3. Register in `orchestrator/router.py`:
   - Add to `SPECIALIST_REGISTRY` dict
   - Add tool enum value to `SPECIALIST_TOOLS[0]["input_schema"]["properties"]["specialist"]["enum"]`

4. Add knowledge docs to `knowledge/your_domain/`

5. Add `evals/scenarios/your_domain_001.yaml` and `your_domain_002.yaml`

6. If the agent introduces a new pattern (new tool, new routing path, new memory contract), update `packages/core/openexecutive/architecture/architecture-facts.yaml`. Pure additions to `SPECIALIST_REGISTRY` are auto-reflected in the `agents` section without YAML edits.

7. Submit PR — must include all of the above

## Company Data

Company-specific data lives in `packages/core/company/` — **gitignored**. Never commit company data. The `.env` file is also gitignored.

Structure:
- `company/profile.yaml` — structured company profile (populated by onboarding wizard)
- `company/docs/` — uploaded documents (indexed into ChromaDB)

## Code Style

- Python: `ruff` for linting, `mypy` for type checking, `pytest` for tests
- Pydantic v2 throughout — use `model_config = ConfigDict(...)` not `class Config`
- All Anthropic API calls: use `anthropic.AsyncAnthropic()`
- No dynamic content in cached system prompt blocks
- All agent `analyze()` calls are async

## Architecture Docs

The `/architecture` page is served from **static, hand-authored content** under `packages/core/openexecutive/architecture/prebuilt/<section_id>.json` — one file per section in `architecture/sections.py` (`SECTIONS`). The backend (`api/routes/architecture.py`) only reads these files; **nothing on this path calls an LLM**. The files ship in the Docker image, so they redeploy automatically with any `packages/core/**` change.

`architecture/architecture-facts.yaml` is the curated, deep source-of-truth reference for the *why* behind the system (integrations, scheduler behavior, departments/people structure, caching layout, invariants, committee review, authority gates). It is **no longer fed to a runtime generator** — treat it as the authoritative notes you (or Claude Code) read when re-authoring a section.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SenteLabsAI/OpenExecutive](https://github.com/SenteLabsAI/OpenExecutive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
