---
trigger: always_on
description: **This project uses GitHub Copilot SDK with Microsoft Agent Framework for security scanning. Read the Python-specific guide at `.vscode/python-copilot-sdk.instructions.md` before writing any agent code.**
---

# AgentSec - AI Agent Coding Guide

**This project uses GitHub Copilot SDK with Microsoft Agent Framework for security scanning. Read the Python-specific guide at `.vscode/python-copilot-sdk.instructions.md` before writing any agent code.**

## Project Architecture

**Monorepo Structure** (3 independent packages in single repo):
```
agentsec/
├── core/                  # Shared agent + skills (Python)
│   └── agentsec/
│       ├── agent.py      # SecurityScannerAgent class
│       ├── config.py     # AgentSecConfig for customization (incl. model selection)
│       ├── orchestrator.py # ParallelScanOrchestrator for concurrent scanning
│       ├── progress.py   # ProgressTracker (uses contextvars.ContextVar)
│       ├── session_runner.py # Shared session-wait logic (run_session_to_completion)
│       ├── session_logger.py # Per-session file logging
│       ├── skill_discovery.py # SCANNER_REGISTRY (single source of truth), classify_files(), dynamic skill discovery
│       ├── tool_health.py # Tool health monitoring, stuck detection, error patterns
│       └── skills.py     # @tool decorated skill functions (LEGACY — see note)
├── cli/                  # Command-line interface (Python)
│   └── agentsec_cli/
│       └── main.py       # argparse CLI entry point
└── desktop/              # GUI + Electron wrapper
    ├── backend/          # FastAPI server (Python)
    │   └── server.py     # HTTP endpoint hosting agent
    └── frontend/         # Next.js UI (TypeScript)
```

> **Note on skills.py**: The `@tool` skill functions (`list_files`, `analyze_file`, `generate_report`) are **legacy MVP code**. The agent now uses Copilot CLI built-in tools (`bash`, `skill`, `view`) as its primary scanning mechanism. These functions are not registered with any SDK session.

**Communication Flow**:
- **CLI**: `main.py` → imports `SecurityScannerAgent` from core → direct agent invocation
- **Desktop**: Electron spawns FastAPI subprocess → FastAPI endpoint exposes agent via HTTP → Next.js frontend calls via SSE

**Deployment**:
- CLI distributed via PyPI: `pip install agentsec`
- Desktop distributed as native installer (Windows NSIS, macOS DMG)

---

## Critical Development Workflows

### Setup (Required First)

```bash
# 1. Verify Python 3.12 (or 3.11 min)
python --version

# 2. Create workspace-level venv (single environment for all 3 packages)
python -m venv venv
.\venv\Scripts\activate  # Windows
source venv/bin/activate # macOS/Linux

# 3. Install Copilot CLI (required for SDK runtime)
copilot --version  # Verify installed
copilot auth login # Authenticate with GitHub

# 4. Install all packages in editable mode
pip install -e ./core        # Agent + skills
pip install -e ./cli         # CLI wrapper
pip install -e ./desktop/backend  # FastAPI server
```

### Agent Development (core/)

```bash
# Define agent in core/agentsec/agent.py
# - SecurityScannerAgent class
# - Uses CopilotClient from SDK
# - Manages session lifecycle (start/initialize/cleanup)
# - Stall detection monitors tool activity; sends nudge messages if LLM stalls
# - Configurable timeout (default 300s) with partial results on timeout
# - Uses Copilot CLI built-in tools (bash, skill, view) for scanning

# Define skills in core/agentsec/skills.py
# - Use @tool decorator from agent_framework
# - Async functions only
# - Return dict with status, results, errors
# - Skills are fallback tools; primary scanning uses Copilot CLI tools

# Test agent invocation
python -c "
import asyncio
from agentsec.agent import SecurityScannerAgent
asyncio.run(SecurityScannerAgent().scan('./test-folder'))
"
```

### CLI Development (cli/)

```bash
# CLI imports agent from core package
# agentsec_cli/main.py uses argparse for command routing
# Key commands: agentsec scan <folder>, agentsec --version

# Test CLI locally
agentsec scan ./test-folder

# Test with custom configuration
agentsec scan ./test-folder --config ./agentsec.yaml
agentsec scan ./test-folder --system-message-file ./custom-prompt.txt

# Test in clean environment before publishing
pip install -e .  # Local install
which agentsec    # Verify command available
agentsec --help   # Verify entry point
```

### Desktop Backend Development (desktop/backend/)

```bash
# FastAPI server exposes agent at /api/scan
# Uses add_agent_framework_fastapi_endpoint() utility
# Implements SSE streaming for real-time progress

# Start server (finds available port automatically)
python desktop/backend/server.py

# Test endpoint
curl -X POST http://localhost:8000/api/scan \
  -H "Content-Type: application/json" \
  -d '{"folder": "./test-folder"}'

# Health check
curl http://localhost:8000/api/health
```

### Desktop Frontend Development (desktop/frontend/)

```bash
# Next.js TypeScript React components
# Key components: FolderSelector, ScanProgress (SSE), ResultsPanel

# Start dev server
cd desktop/frontend && npm run dev

# Connect to backend (configure via env var)
NEXT_PUBLIC_API_URL=http://localhost:8000

# Test end-to-end: select folder → click scan → stream results
```

---

## Configuration System


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alxayo/sec-check](https://github.com/alxayo/sec-check) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
