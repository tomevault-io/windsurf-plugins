---
trigger: always_on
description: > This file helps AI coding agents (GitHub Copilot, Copilot Workspace, Codespaces agents, etc.) understand and work effectively with this repository.
---

# Agents Guide — Architecture Review Agent Sample

> This file helps AI coding agents (GitHub Copilot, Copilot Workspace, Codespaces agents, etc.) understand and work effectively with this repository.

---

## Repository Purpose

This is an **AI-powered Architecture Review Agent** that analyses software architecture descriptions and generates interactive diagrams. It accepts any input format (YAML, Markdown, plaintext, code, design docs, prose) and returns structured risk assessments, Excalidraw diagrams, PNG exports, and actionable recommendations.

---

## Project Structure

```
agent-architecture-review-sample/
├── main.py              # Hosted agent entry point (Microsoft Agent Framework / Foundry)
├── api.py               # FastAPI backend (REST API for the web UI)
├── tools.py             # Core engine — all tool logic (parser, risk detector, diagram renderer, MCP client, PNG export)
├── run_local.py         # CLI runner for local testing (no Azure required for structured inputs)
├── agent.yaml           # Foundry hosted agent deployment manifest
├── requirements.txt     # Python dependencies (pinned versions)
├── pyproject.toml       # Pytest configuration
├── Dockerfile           # Container for hosted agent deployment
├── Dockerfile.web       # Container for web app deployment (FastAPI + React)
├── docs/deployment.md   # Step-by-step deployment & RBAC guide
├── frontend/            # React UI (Vite + Excalidraw)
│   ├── src/App.jsx      # Main React app with input, tabs, results
│   ├── src/api.js       # API client — calls FastAPI backend
│   └── src/components/  # Summary, RiskTable, ComponentMap, DiagramViewer, Recommendations
├── scenarios/           # Pre-built demo architecture files (YAML, Markdown)
├── tests/               # Pytest test suite
│   ├── conftest.py      # Shared fixtures (sample components, connections)
│   ├── test_tools.py    # Unit tests for tools.py functions
│   ├── test_api.py      # FastAPI endpoint tests
│   └── test_integration.py  # Integration tests
├── scripts/             # Automation scripts
│   ├── windows/         # PowerShell: setup.ps1, dev.ps1, deploy-webapp.ps1, teardown.ps1
│   └── linux-mac/       # Bash: setup.sh, dev.sh, deploy-webapp.sh, teardown.sh
└── output/              # Generated outputs (auto-created): .excalidraw, .png, .json
```

---

## Key Files and Responsibilities

| File | Role | When to modify |
|------|------|----------------|
| `tools.py` | **Core engine** — parser, LLM inference, risk detection, Excalidraw/PNG rendering, component mapping, report builder | Adding analysis capabilities, new parsers, new risk detectors, diagram improvements |
| `main.py` | Hosted agent entry point — registers tools with Microsoft Agent Framework, defines agent instructions | Changing agent behaviour, adding/removing tools, modifying system prompt |
| `api.py` | FastAPI REST API — exposes review pipeline as HTTP endpoints, serves React frontend in production | Adding/modifying API endpoints, changing request validation, CORS config |
| `run_local.py` | CLI test runner — runs the full pipeline locally with Rich console output | Improving CLI experience, adding CLI flags |
| `agent.yaml` | Foundry deployment manifest — defines agent name, model, protocols, environment variables | Changing deployment config, model, environment variables |
| `requirements.txt` | Python dependencies — pinned versions | Adding/updating dependencies |

---

## Architecture and Data Flow

```
Input (any format) → smart_parse() → [rule-based OR LLM fallback]
    → analyze_risks() → generate_excalidraw_elements() → export_png()
    → build_component_map() → build_review_report()
```

### Core Pipeline (`tools.py`)

1. **`smart_parse(content)`** — Tries rule-based parsing first (YAML → `_parse_yaml`, Markdown → `_parse_markdown`, text → `_parse_text`). If it extracts ≤1 component, automatically falls back to `infer_architecture_llm()`.
2. **`analyze_risks(components, connections)`** — Template-based risk detection across 4 categories: SPOF, scalability, security, anti-patterns. LLM-inferred inputs carry their own risks.
3. **`generate_excalidraw_elements()`** — Builds Excalidraw JSON with layered layout (frontend → gateway → service → database).
4. **`export_png()`** — Renders PNG using Pillow with the same layout engine.
5. **`build_component_map()`** — Dependency analysis with fan-in/fan-out metrics.
6. **`build_review_report()`** — Composes executive summary, risk assessment, component map, diagram info, and prioritised recommendations.

### Two Deployment Paths

- **Option A — Web App**: `api.py` (FastAPI) + `frontend/` (React). Deployed to Azure App Service via `Dockerfile.web`.
- **Option B — Hosted Agent**: `main.py` (Agent Framework). Deployed to Microsoft Foundry via `Dockerfile` and VS Code Foundry extension (v2 hosted agent services).

Both share the same core engine in `tools.py`.

---

## Technology Stack

- **Python 3.11+**
- **Microsoft Agent Framework** (`azure-ai-agentserver-agentframework` v1.0.0b12) — hosted agent runtime (v2 hosted agent services)
- **Azure OpenAI** (GPT-4.1) — LLM backend via `AzureOpenAIChatClient`
- **FastAPI** + **Uvicorn** — REST API

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure-Samples/agent-architecture-review-sample](https://github.com/Azure-Samples/agent-architecture-review-sample) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
