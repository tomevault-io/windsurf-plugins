---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Project Overview
This is a multimodal agentic AI framework for proteomics laboratory work that uses Google's Agent Development Kit (ADK), 
Gemini models, and Vertex AI. The system captures and shares practical expertise by linking written instructions to 
real-world laboratory work through video analysis.

## Architecture

### Root Agent and Sub-Agents

The project uses a hierarchical agent architecture with a root orchestrator and specialized sub-agents:

- **Root Agent** (`proteomics_lab_agent/agent.py`): Orchestrates all sub-agents and provides common tools like datetime functions
- **Sub-Agents** (`proteomics_lab_agent/sub_agents/`):
  - `protocol_generator_agent`: Analyzes video/audio to generate formatted protocols
  - `lab_note_generator_agent`: Compares researcher actions on video against reference protocols
  - `lab_knowledge_agent`: Retrieves documents from Confluence via MCP server
  - `instrument_agent`: Monitors mass spectrometer performance via AlphaKraken MCP server
  - `qc_memory_agent`: Logs QC ratings using local SQLite database via MCP server
  - `video_analyzer_agent`: Video analysis utilities

### Sub-Agent Structure

Each sub-agent follows this pattern:
```
sub_agents/<agent_name>/
├── __init__.py          # Exports the agent
├── agent.py             # Agent definition with ADK Agent/LlmAgent
├── prompt.py            # Agent-specific prompt/instructions
└── [additional modules] # Agent-specific utilities
```

### MCP Server Integration

The project uses MCP (Model Context Protocol) servers for external integrations:

- **AlphaKraken MCP**: Proteomics analysis platform integration (Docker container)
- **Confluence MCP**: Knowledge management system integration (Docker container)
- **QC Memory MCP**: Local SQLite database server (`qc_memory_agent/server.py`)

MCP servers are integrated via:
- `MCPToolset` with `StreamableHTTPServerParams` for HTTP-based servers (AlphaKraken, Confluence)
- `MCPToolset` with `StdioServerParameters` for local stdio-based servers (QC Memory)

### Database Schema

The QC Memory agent uses SQLite with schema versioning:
- Database location: `proteomics_lab_agent/sub_agents/qc_memory_agent/database.db`
- Schema management: `create_db.py` creates database, `db_interface.py` provides CRUD operations
- Schema version checking enforced on every connection (see `get_db_connection()` in `db_interface.py`)
- Compatible schema version defined by `COMPATIBLE_SCHEMA_VERSION` constant

## Development Commands

### Environment Setup

1. Create and activate virtual environment:
```bash
python3 -m venv .venv
source .venv/bin/activate  # macOS/Linux
.venv\Scripts\activate     # Windows
```

2. Install dependencies:
```bash
pip install -r requirements/requirements.txt
pip install -r requirements/requirements_development.txt  # for development
```

3. Authenticate with Google Cloud:
```bash
gcloud auth login
gcloud init
```

4. Configure environment variables:
   - Copy `.env.example` to `.env` and fill in values
   - Copy `.env.secrets.example` to `.env.secrets` and fill in secrets

### Running the Agent

Start MCP server containers (from project root):
```bash
docker compose --env-file ./.env.secrets --env-file ./.env up confluence_mcp alphakraken_mcp
```

Run the agent (in separate terminal):
```bash
adk run proteomics_lab_agent              # CLI interface
adk web                                    # Web UI (local only)
adk web --host 0.0.0.0                    # Web UI (accessible on network)
```

Or run via Docker Compose (full deployment):
```bash
docker compose --env-file ./.env.secrets --env-file ./.env up
```

### Testing and Code Quality

Run pre-commit hooks:
```bash
pre-commit run --all-files
```

Individual checks:
```bash
ruff format .                    # Format code
ruff check --fix .               # Lint and auto-fix
detect-secrets scan --exclude-files testfiles --exclude-lines '"(hash|id|image/\w+)":.*' > .secrets.baseline
detect-secrets audit .secrets.baseline
```

### Docker Operations

Build containers:
```bash
docker compose --env-file ./.env.secrets --env-file ./.env build
```

Start in production (detached):
```bash
docker compose --env-file ./.env.secrets --env-file ./.env up -d
```

Stop containers:
```bash
docker container stop python_lab_agent alphakraken_mcp confluence_mcp
```

### Database Operations

Check QC Memory database (requires sqlite3):
```bash
sqlite3 proteomics_lab_agent/sub_agents/qc_memory_agent/database.db
```

## Configuration

### Model Configuration

Models are configured in `proteomics_lab_agent/config.py`:
- `model`: Default model for generation tasks (currently "gemini-2.5-flash")
- `analysis_model`: Model for video analysis (currently "gemini-2.5-pro")
- `temperature`: Controls output randomness (default 0.9)

### Environment Variables

Key environment variables in `.env`:
- `GOOGLE_CLOUD_PROJECT`: GCP project ID
- `GOOGLE_CLOUD_STORAGE_BUCKET`: Bucket for video storage
- `ALPHAKRAKEN_MCP_URL`: AlphaKraken MCP server URL (differs between local/docker)
- `CONFLUENCE_MCP_URL`: Confluence MCP server URL (differs between local/docker)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MannLabs/proteomics_lab_agent](https://github.com/MannLabs/proteomics_lab_agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
