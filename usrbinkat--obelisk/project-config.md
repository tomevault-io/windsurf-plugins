---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
Obelisk is a tool that transforms Obsidian vaults into MkDocs Material Theme sites with AI integration through Ollama and Open WebUI. It preserves Obsidian's rich features while delivering a modern documentation website with AI chat capabilities, including a RAG (Retrieval-Augmented Generation) pipeline for context-aware responses.

## Repository
- GitHub: https://github.com/usrbinkat/obelisk

## Core Features
- Convert Obsidian vaults to MkDocs Material Theme sites (wiki links, callouts, comments)
- Built-in AI chatbot integration via Ollama and Open WebUI
- RAG pipeline for context-aware document retrieval
- Vector database integration (ChromaDB with Milvus planned)
- Custom styling and theming capabilities
- Documentation versioning with mike
- Docker and container-based deployment

## Architecture

The project consists of several integrated components:

1. **Python Package** (`/obelisk/`): Core utilities
   - `cli.py`: Command-line interface
   - `config.py`: Configuration management
   - `convert.py`: Obsidian to MkDocs conversion
   - `/rag/`: RAG system components
     - `service.py`: Main RAG service coordinator
     - `document.py`: Document processing and chunking
     - `embedding.py`: Vector embedding generation
     - `storage.py`: Vector database integration
     - `api.py`: OpenAI-compatible API endpoints

2. **Documentation Content** (`/vault/`): Source content and customizations
   - `stylesheets/extra.css`: Custom CSS styles
   - `javascripts/extra.js`: Custom JavaScript
   - `overrides/main.html`: HTML template overrides

3. **Container Architecture**:
   - `obelisk`: MkDocs documentation server (port 8000)
   - `ollama`: Local LLM serving (port 11434)
   - `open-webui`: Chat interface (port 8080)
   - `obelisk-rag`: RAG API service (port 8001)
   - `milvus`: Vector database (port 19530)
   - `litellm`: LLM API proxy for integration
   - `init-service`: Container initialization

## Project Structure
```
/workspaces/obelisk/
├── obelisk/                # Python package
│   ├── __init__.py         # Version and metadata
│   ├── cli.py              # Command-line interface
│   ├── config.py           # Configuration utilities
│   └── convert.py          # Conversion logic
├── vault/                  # Documentation content
│   ├── assets/             # Static assets
│   ├── stylesheets/        # CSS customizations
│   ├── javascripts/        # JS customizations
│   └── overrides/          # HTML template overrides
├── .github/workflows/      # CI/CD configuration
├── mkdocs.yml              # MkDocs configuration
├── docker-compose.yaml     # Docker services definition
├── Dockerfile              # Container definition
├── pyproject.toml          # Python project definition
└── README.md               # Project documentation
```

## Build Commands
- `poetry install --no-root` - Install dependencies
- `poetry run mkdocs build` - Build static site
- `poetry run mkdocs build --clean --strict` - Run strict build testing
- `poetry run mkdocs serve --livereload --dirty` - Fast development server with livereload
- `poetry run mkdocs serve --livereload --watch-theme --open` - Build and serve with browser opening
- `rm -rf site && find . -type d -name __pycache__ -exec rm -rf {} +` - Remove build artifacts
- `poetry update` - Update all dependencies
- `poetry run mkdocs gh-deploy --force` - Deploy to GitHub Pages
- `poetry run mike deploy --push --update-aliases <VERSION> "<DESCRIPTION>"` - Deploy version
- `poetry run mike set-default --push <VERSION>` - Set default version

## Docker Commands
- `docker build -t obelisk:latest .` - Build Docker container
- `docker run -p 8000:8000 -v ${PWD}/vault:/app/vault -v ${PWD}/mkdocs.yml:/app/mkdocs.yml -v ${PWD}/pyproject.toml:/app/pyproject.toml -v ${PWD}/poetry.lock:/app/poetry.lock obelisk:latest` - Run with local volumes mounted
- `docker-compose up obelisk` - Run Obelisk service only
- `docker-compose up` - Run full stack with Ollama and OpenWebUI

## Creating New Content
To create a new markdown file in the vault:
```bash
mkdir -p vault
cat > vault/page-name.md << EOF
---
title: page-name
date: $(date +%Y-%m-%d)
---

EOF
```

## AI Integration

Obelisk integrates AI capabilities through a multi-service architecture:

1. **Ollama**: Lightweight local model server
   - Port: `11434`
   - Supports models like Llama3, Mistral, Phi, Gemma
   - Used for both text generation and embeddings

2. **Open WebUI**: Web interface for chat
   - Port: `8080`
   - Connected to Ollama for model inference
   - Supports direct RAG integration

3. **Documentation Server**: MkDocs site
   - Port: `8000`
   - Integrated with chat UI via JavaScript

4. **RAG API Service**: OpenAI-compatible API
   - Port: `8001`
   - Processes and indexes markdown documents
   - Handles document retrieval via vector search
   - Augments LLM responses with relevant context

5. **LiteLLM Proxy**: LLM middleware
   - Port: `4000`
   - Provides unified interface for multiple LLM providers
   - Handles authentication and routing

The RAG pipeline flow:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [usrbinkat/obelisk](https://github.com/usrbinkat/obelisk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
