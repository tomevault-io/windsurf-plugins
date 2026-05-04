---
trigger: always_on
description: This repository powers the LlamaFarm CLI + server for retrieval-augmented AI workflows. Automated tools that reason about or modify the codebase should follow the guidance below. Think of this file as the “living handbook” for LLM coding agents.
---

# LlamaFarm Repository Guidelines for Cursor & Automated Coding Agents

This repository powers the LlamaFarm CLI + server for retrieval-augmented AI workflows. Automated tools that reason about or modify the codebase should follow the guidance below. Think of this file as the “living handbook” for LLM coding agents.

---

## 1. High-Level Overview
- **Primary goal:** Provide a CLI (`lf`) and API server for configuring projects, ingesting datasets, and running RAG-enhanced chat sessions.
- **Stack:**
  - **CLI:** Go (Cobra) under `cli/`
  - **Server:** Python (FastAPI) under `server/`
  - **RAG worker:** Python/Celery under `rag/`
  - **Docs site:** Docusaurus (Node/Nx) under `docs/website/`
- **Configuration:** Driven by `llamafarm.yaml`, defined via JSON Schema in `config/schema.yaml` and `rag/schema.yaml`.
- **Extensibility:** Designed to allow new runtime providers, vector stores, parsers, extractors, and CLI commands.

Agents must keep documentation and code aligned—any change to workflows or schema requires updates across README + docs.

---

## 2. Prerequisites & Installation
### Required Software
1. **Docker** – `lf start` spins up the API and Celery worker automatically via Docker.
2. **Ollama** – current default runtime (download from https://ollama.com/download). Additional providers will follow the OpenAI-compatible API pattern.
3. **CLI (lf)** – install first; sources are at the repo root, but the standard install is via script.

### CLI Installation (macOS/Linux)
```bash
curl -fsSL https://raw.githubusercontent.com/llama-farm/llamafarm/main/install.sh | bash
```
- Script auto-detects platform, downloads latest binary, and installs to `/usr/local/bin` by default. Accepts `--install-dir` and `--version` flags if customization is needed.

### CLI Installation (Windows)
- Download `lf.exe` from https://github.com/llama-farm/llamafarm/releases/latest and add it to your PATH.

### Optional Development Stack Installations
If you need to run components manually (instead of through the CLI):
- **Python 3.10+**
- **uv** package manager (`curl -LsSf https://astral.sh/uv/install.sh | sh`)
- **Go 1.24+** (for building CLI from source or in forks)
- **Node.js + pnpm** (for doc site maintenance)

---

## 3. Quickstart Commands
1. **Install CLI** (see above).
2. **Increase Ollama context window** – open the Ollama app → Settings → Advanced → set context window to match production (e.g., 100k tokens). Larger context windows improve RAG results on long documents.
3. **Create a project & launch services**
   ```bash
   lf init my-project
   lf start
   ```
   - `lf start` launches the FastAPI app and Celery worker via Docker, and opens an interactive chat TUI. Exit with `Ctrl+C`.
4. **Ingest and query**
   ```bash
   lf datasets create -s pdf_ingest -b main_db research
   lf datasets upload research ./examples/fda_rag/files/*.pdf
   lf datasets process research
   lf rag query --database main_db "Which letters mention clinical trial data?"
   lf chat "Summarize the 2024 FDA letters"
   ```

---

## 4. Repository Layout (top-level)
| Path | Purpose |
|------|---------|
| `README.md` | Developer quickstart and extensibility overview.
| `config/` | JSON Schema (`schema.yaml`), generated Pydantic/Go types (`datamodel.py`, `config_types.go`).
| `server/` | FastAPI application and services; entrypoint `server/main.py`.
| `rag/` | Celery worker, ingestion pipeline, parser implementations.
| `cli/` | Go CLI; commands under `cli/cmd/`, entry `main.go`.
| `docs/website/` | Docusaurus docs (run via Nx).
| `examples/` | Ready-made example configs (FDA, Raleigh UDO).
| `.claude/` | LLM assistant instructions (this document).
| `.agents/` | Research/plan workflow guidelines for coding agents.

---

## 5. Running Components Locally
### Using the CLI (Recommended)
- `lf start` – orchestrates Docker containers and starts the dev chat UI.
- `lf datasets create|upload|process` – manage ingestion.
- `lf rag query`, `lf rag health`, `lf rag stats` – retrieval & diagnostics.
- `lf chat` – send single prompts (supports RAG toggles and curl preview).

### Manual Server & Worker (when needed)
```bash
# API server
cd server
uv sync
uv run uvicorn server.main:app --reload --host 0.0.0.0 --port 8000

# RAG worker
cd ../rag
uv sync
uv run python cli.py worker
```
- Normally not required because `lf start` handles this in Docker. Manual control is useful for debugging without Docker.

---

## 6. Building the CLI from Source (including forks)
```bash
# clone the repo (fork or upstream)
git clone https://github.com/llama-farm/llamafarm.git
cd llamafarm/cli

# install dependencies if needed
go mod tidy

# build the binary
go build -o lf main.go

# optional: place in PATH
sudo mv lf /usr/local/bin/
```
- Dev mode: `go run main.go [command]`.
- Tests: `go test ./...`.
- After schema changes (local or forked), regenerate Go types via `config/generate_types.py` to keep `config_types.go` aligned.

---

## 7. Configuration (`llamafarm.yaml`)
- Schema: `config/schema.yaml`
- RAG specifics: `rag/schema.yaml`

### Required Fields
| Field | Notes |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [llama-farm/llamafarm](https://github.com/llama-farm/llamafarm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
