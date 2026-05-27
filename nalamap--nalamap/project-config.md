---
trigger: always_on
description: > **Target Audience**: AI coding assistants (Cursor, Copilot, etc.) and automated development tools.
---

# AGENTS.md - AI Development Context

> **Target Audience**: AI coding assistants (Cursor, Copilot, etc.) and automated development tools.

This document provides high-level context and specific instructions for AI agents working on the NaLaMap project. For standard development procedures, please refer to **[CONTRIBUTING.md](CONTRIBUTING.md)**.

---

## 🤖 Core Directives for Agents

When contributing to this repository, you **MUST** follow these rules:

1.  **Single Source of Truth**: All development workflows (testing, linting, running) are defined in **[CONTRIBUTING.md](CONTRIBUTING.md)**. Do not hallucinate alternative procedures.
2.  **Test-Driven Development**:
    *   **Always** run existing tests before modifying code to establish a baseline.
    *   **Always** write new tests for new features or bug fixes.
    *   **Verify** changes by running the relevant test suite (`pytest` for backend, `playwright` for frontend).
3.  **Code Quality**:
    *   **Linting is mandatory**. After editing files, run the linters defined in `CONTRIBUTING.md` (`flake8`, `black`, `npm run lint`).
    *   Fix linter errors immediately. Do not submit code with linting violations.
4.  **Documentation**:
    *   Update `README.md` files if you change how a component works.
    *   If you add a new tool, update `backend/services/tools/README.md`.
    *   If you add a new dependency, update `pyproject.toml` or `package.json`.

---

## 🗺️ Repository Map & Context

To help you navigate the codebase efficiently:

### Backend Structure (`backend/`)
*   **API Layer**: `api/` (FastAPI endpoints).
*   **Business Logic**: `services/` (AI agents, tools, database logic).
    *   `services/agents/`: Legacy agent implementations.
    *   `services/tools/`: **Core AI tools** (geocoding, geoprocessing). **Read `backend/services/tools/README.md`** for details.
*   **Data Models**: `models/` (Pydantic models).
*   **Tests**: `tests/` (pytest suite). **Read `backend/tests/README.md`** for details.

### Frontend Structure (`frontend/`)
*   **Pages**: `app/page.tsx` (Main entry).
*   **Components**: `app/components/` (React components).
    *   `maps/`: Leaflet map logic.
    *   `chat/`: AI chat interface.
*   **State**: `app/stores/` (Zustand stores).
*   **Tests**: `tests/` (Playwright E2E). **Read `frontend/tests/README.md`** for details.

### Key Configuration Files
*   `backend/pyproject.toml`: Python dependencies and tool config (flake8, black).
*   `frontend/package.json`: Node dependencies and scripts.
*   `docker-compose.yml`: Production deployment.
*   `dev.docker-compose.yml`: Development environment.

---

## 🛠️ Common Tasks for Agents

### 1. Adding a New AI Tool
When asked to add a new capability to the AI assistant:
1.  Create the tool function in `backend/services/tools/`.
2.  Use the `@tool` decorator from `langchain.tools`.
3.  Add unit tests in `backend/tests/`.
4.  **Register the tool** in `backend/services/default_agent_settings.py`.
5.  Document it in `backend/services/tools/README.md`.

### 2. Modifying the Agent Workflow
The main agent logic is in `backend/services/single_agent.py` (LangGraph ReAct agent).
*   If changing the system prompt, look for `DEFAULT_SYSTEM_PROMPT`.
*   If changing tool selection logic, check `create_geo_agent`.

### 3. Debugging
If tests fail:
*   **Backend**: Read the `pytest` output. Use `pytest -vv` for more detail. Check `backend/app_output.log` if available.
*   **Frontend**: Use `npx playwright test --ui` or check the HTML report.

---

## 🔄 Quick Reference: Commands

**Critical Checklist**:

✅ **All tests pass**:
```bash
cd backend && poetry run pytest tests/
cd frontend && npm test
```

✅ **Code is properly formatted**:
```bash
cd backend && poetry run black .
cd backend && poetry run flake8 .
```

✅ **No regressions introduced** (run relevant test suites)

✅ **New features have tests** (aim for >80% coverage)

✅ **Documentation updated** (if adding new features or changing APIs)

### Committing Changes

```bash
# Stage changes
git add .

# Commit with descriptive message
git commit -m "feat: Add new geocoding functionality"

# Push to your branch
git push origin features/YYYYMMDD_YourFeatureName
```

### Creating Pull Requests

1. Push your branch to GitHub
2. Create a Pull Request targeting `main`
3. Ensure CI/CD checks pass (see [CI/CD Pipeline](#cicd-pipeline))
4. Request review from maintainers
5. Address review feedback
6. Merge once approved

---

## 🛠️ Common Development Tasks

### Adding a New Backend Dependency

```bash
cd backend
poetry add <package-name>

# For dev dependencies
poetry add --group dev <package-name>
```

### Adding a New Frontend Dependency

```bash
cd frontend
npm install <package-name>

# For dev dependencies
npm install --save-dev <package-name>
```

### Adding a New API Endpoint

1. **Create/modify endpoint in** `backend/api/`
2. **Add corresponding tests in** `backend/tests/`
3. **Update models if needed in** `backend/models/`
4. **Update frontend API calls in** `frontend/app/` (components, hooks, or stores)
5. **Add frontend E2E tests in** `frontend/tests/`
6. **Run full test suite**

### Adding a New AI Tool

1. **Create tool in** `backend/services/tools/`
2. **Add tool tests in** `backend/tests/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nalamap/nalamap](https://github.com/nalamap/nalamap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
