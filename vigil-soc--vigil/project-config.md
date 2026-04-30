---
trigger: always_on
description: This file provides guidance for AI assistants (Claude Code and similar tools) working in this repository.
---

# CLAUDE.md — Vigil SOC

This file provides guidance for AI assistants (Claude Code and similar tools) working in this repository.

---

## Project Overview

**Vigil** is an open-source, AI-native Security Operations Center (SOC) platform. It orchestrates 12 specialized AI agents via Claude to perform triage, investigation, threat hunting, forensics, and automated response across 30+ security integrations.

**Core pillars:**
- **Agents** — 12 specialized AI agents (Triage, Investigator, Threat Hunter, Correlator, Responder, Reporter, MITRE Analyst, Forensics, Threat Intel, Compliance, Malware Analyst, Network Analyst)
- **Workflows** — Multi-agent orchestrated playbooks (Incident Response, Full Investigation, Threat Hunt, Forensic Analysis)
- **Integrations** — 30+ tools via MCP protocol (Splunk, CrowdStrike, VirusTotal, Shodan, Timesketch, Jira, Slack, etc.)

**Ports:**
- Backend API: `http://localhost:6987`
- Frontend: `http://localhost:6988`
- PostgreSQL: `5432`
- Redis: `6379`

---

## Repository Structure

```
vigil/
├── backend/              # FastAPI REST API
│   ├── main.py           # App entry point, router registration
│   ├── api/              # 27 route modules (findings, cases, claude, auth, etc.)
│   ├── middleware/       # Auth middleware
│   └── schemas/          # Pydantic request/response schemas
├── services/             # 40+ business logic service classes
│   ├── claude_service.py # Central AI orchestration (largest file ~124KB)
│   ├── soc_agents.py     # Agent prompt definitions
│   ├── mcp_service.py    # MCP server coordination
│   └── case_*_service.py # Case lifecycle services
├── daemon/               # Autonomous 24/7 SOC background process
│   ├── orchestrator.py   # Main autonomous agent orchestrator
│   ├── agent_runner.py   # Executes agents with cost/resource guardrails
│   ├── poller.py         # Fetches alerts from SIEM/EDR
│   ├── processor.py      # Processes findings through AI pipeline
│   ├── responder.py      # Executes containment actions
│   └── scheduler.py      # Cron-style scheduled tasks
├── frontend/             # React + TypeScript + Vite SPA
│   └── src/
│       ├── pages/        # Route-level page components
│       ├── components/   # Feature components (16 subdirectories)
│       ├── services/     # Axios API client services
│       ├── contexts/     # React Context (auth, theme)
│       └── theme/        # MUI customization
├── workflows/            # Workflow definitions as WORKFLOW.md files
│   ├── incident-response/WORKFLOW.md
│   ├── full-investigation/WORKFLOW.md
│   ├── threat-hunt/WORKFLOW.md
│   └── forensic-analysis/WORKFLOW.md
├── tools/                # MCP tool implementations (15+ integrations)
├── mcp-servers/          # Git submodule: MCP server implementations
├── deeptempo-core/       # Git submodule: core AI/detection library
├── database/
│   └── init/             # PostgreSQL init SQL (runs in order 01-06)
├── core/                 # Config, secrets management, rate limiting
├── data/                 # Schemas, MITRE taxonomy, detection registry
├── tests/                # pytest + vitest test suites
├── docs/                 # Detailed documentation
├── docker/               # docker-compose.yml + Dockerfiles
├── scripts/              # Init and utility shell scripts
├── mcp-config.json       # 30+ MCP server definitions
└── env.example           # Template for all 220+ environment variables
```

---

## Development Setup

### Quick Start

```bash
git clone --recurse-submodules https://github.com/Vigil-SOC/vigil.git
cd vigil
./start_web.sh       # Starts PostgreSQL (Docker), backend, and frontend
```

### Manual Start

```bash
# 1. Start infrastructure
cd docker && docker compose up -d postgres redis

# 2. Backend (from repo root)
source venv/bin/activate
uvicorn backend.main:app --host 0.0.0.0 --port 6987 --reload

# 3. Frontend
cd frontend && npm run dev

# 4. (Optional) Daemon
./start_daemon.sh
```

### Fresh Environment

```bash
./setup_dev.sh   # Creates venv, installs all Python + npm deps
```

### Prerequisites

- **Python 3.10+** (required by claude-agent-sdk)
- **Node.js 18+**
- **Docker Desktop** (must be running — used for PostgreSQL and Redis)
- **Git** with submodule support

### Environment Variables

Copy `env.example` to `.env` and populate as needed. Key variables:

| Variable | Purpose | Default |
|----------|---------|---------|
| `DEV_MODE` | Bypass all authentication | `true` |
| `ANTHROPIC_API_KEY` | Claude API access | required for AI features |
| `DATABASE_URL` | PostgreSQL connection | auto-set by docker-compose |
| `REDIS_URL` | ARQ job queue | `redis://localhost:6379/0` |
| `SPLUNK_URL` / credentials | Splunk SIEM | optional |
| `CROWDSTRIKE_CLIENT_ID/SECRET` | CrowdStrike EDR | optional |
| `VIRUSTOTAL_API_KEY` | Threat intel | optional |

Default dev login: **admin / admin123** (when `DEV_MODE=false`)

---

## Running Tests

### Python (pytest)

```bash
# All tests
pytest

# With coverage
pytest --cov=. --cov-report=html

# By marker
pytest -m unit
pytest -m integration   # requires running PostgreSQL
pytest -m "not slow"

# Specific file
pytest tests/test_backend_tools.py -v
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Vigil-SOC/vigil](https://github.com/Vigil-SOC/vigil) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
