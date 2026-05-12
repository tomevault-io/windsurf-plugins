---
trigger: always_on
description: Contelligence is an AI-native, agentic content intelligence platform powered by the GitHub Copilot SDK. It replaces brittle content processing pipelines with autonomous AI agents that ingest content from any source/format, understand data by meaning, and deliver structured intelligence — all orchestrated through natural language.
---

# AGENTS.md

Contelligence is an AI-native, agentic content intelligence platform powered by the GitHub Copilot SDK. It replaces brittle content processing pipelines with autonomous AI agents that ingest content from any source/format, understand data by meaning, and deliver structured intelligence — all orchestrated through natural language.

## Project structure

This is a three-service monorepo deployed via Azure Developer CLI (`azd`).

- `contelligence-agent/` — Python/FastAPI backend (REST API, agent orchestration, Azure connectors)
- `contelligence-web/` — React/TypeScript frontend (SPA with Vite, Tailwind, Shadcn/ui)
- `contelligence-cowork/` — Electron desktop app (bundles the Python backend as a native macOS/Windows/Linux app)
- `infra/` — Azure infrastructure-as-code (Bicep)
- `docs/` — Architecture docs, specs, and plans
- `.github/skills/` — Copilot skills (code-free domain knowledge)
- `azure.yaml` — Azure Developer CLI service definitions

For backend internals, see [contelligence-agent/AGENTS.md](contelligence-agent/AGENTS.md).
For frontend internals, see [contelligence-web/AGENTS.md](contelligence-web/AGENTS.md).
For desktop app internals, see [contelligence-cowork/AGENTS.md](contelligence-cowork/AGENTS.md).

## Setup commands

```bash
# Backend — install deps and start dev server
cd contelligence-agent
pip install -r requirements.txt
uvicorn main:app --reload --port 8081

# Frontend — install deps and start dev server
cd contelligence-web
npm install
npm run dev

# Desktop app — install deps and start dev server
cd contelligence-cowork
npm install
npm start

# Desktop app — build distributable (requires PyInstaller backend first)
./scripts/build-cowork.sh            # both backend + desktop
./scripts/build-cowork.sh backend    # PyInstaller binary only
./scripts/build-cowork.sh cowork     # Electron package only

# Full stack — Docker Compose (includes Copilot CLI + Azure MCP sidecar)
docker compose -f docker-compose.agent.yml up --build

# Deploy to Azure
azd up
```

## Testing commands

```bash
# Backend — run all tests
cd contelligence-agent && python -m pytest

# Backend — run a single test file
python -m pytest tests/unit/test_tool_registry.py

# Backend — run a single test by name
python -m pytest tests/unit/test_tool_registry.py -k "test_register_and_get_tool"

# Backend — run tests by category
python -m pytest tests/unit/
python -m pytest tests/integration/
python -m pytest tests/e2e/
python -m pytest tests/behavioral/
python -m pytest tests/smoke/

# Frontend — run tests
cd contelligence-web && npm test

# Frontend — lint
cd contelligence-web && npm run lint
```

Always run relevant tests after modifying code. Use file-scoped test runs, not full suite runs.

## Code style

### Do

- Use Python 3.12+ features (type unions with `|`, match statements where appropriate)
- Use `async`/`await` throughout the backend — all I/O is async
- Use Pydantic `BaseModel` for all request/response schemas and data models
- Use FastAPI dependency injection via `Depends()` — never instantiate services in routers
- Use the connector abstraction layer (`app/connectors/`) for all Azure service calls
- Use the service layer (`app/services/`) for business logic — routers stay thin
- Use Tailwind CSS utility classes and Shadcn/ui components in the frontend
- Use React Query (`@tanstack/react-query`) for server state management
- Use `zod` schemas for frontend form validation
- Use path aliases (`@/components/...`, `@/lib/...`) in frontend imports
- Use `DefaultAzureCredential` for all Azure authentication — no hardcoded keys
- Keep components small and focused — one concern per file
- Keep diffs small and focused — avoid repo-wide rewrites unless explicitly asked
- Write tests for new code paths — prefer unit tests in `tests/unit/`
- Add or update tests when fixing bugs — add a failing test first, then fix to green

### Don't

- Don't use synchronous I/O in the backend — everything must be `async`
- Don't hardcode Azure connection strings, keys, or secrets — use environment variables via `AppSettings`
- Don't fetch data directly in React components — use React Query hooks or `lib/api.ts`
- Don't use inline styles in the frontend — use Tailwind utilities or CSS variables
- Don't add heavy new dependencies without discussion
- Don't bypass the connector/service layer to make direct Azure SDK calls from routers
- Don't use `print()` — use structured logging via the observability module
- Don't commit `.env` files, secrets, or credentials

## Domain concepts

- **Session** — a conversation between a user and the agent; has turns, tool calls, and output artifacts
- **Agent** — a configurable persona with a system prompt, tool set, and optional skills (custom agents are stored in Cosmos DB)
- **Skill** — a code-free domain knowledge package (SKILL.md format) that extends agent capabilities
- **Schedule** — a cron/interval/webhook trigger that automatically runs an agent with a given instruction

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nadeemis/contelligence](https://github.com/nadeemis/contelligence) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
