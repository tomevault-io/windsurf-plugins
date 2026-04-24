---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Adaptation Atlas Co-Pilot - A climate adaptation AI assistant with two main components:

1. **Frontend**: React 19 + TypeScript + Vite application providing a modern web interface
2. **Backend**: LangGraph-based AI agent that generates visualizations and text summaries of climate adaptation data

The system uses Mistral AI models for chat and code generation, with both a Chainlit interface for development and a React frontend for production use.

## Setup

### Backend Setup

```bash
# Initial setup
git clone git@github.com:AdaptationAtlas/adaptation-atlas-assistant.git
cd adaptation-atlas-assistant
uv sync
uv run prek install

# Configure environment
cp .env.example .env
# Add MISTRAL_API_KEY and set CHAT_MODEL_SIZE (large/medium/small)

# Initialize dataset embeddings (required before first run)
uv run python scripts/embed_datasets.py

# Run the Chainlit UI (for development)
uv run chainlit run chainlit/app.py -w
```

### Frontend Setup

```bash
# Navigate to frontend directory
cd frontend

# Install dependencies
npm install

# Start development server
npm run dev  # Runs on http://localhost:5173

# Build for production
npm run build
```

Work is tracked on the [project board](https://github.com/orgs/developmentseed/projects/158).

## Essential Commands

### Backend Commands
```bash
# Run all tests (unit tests only)
uv run pytest

# Run tests including agent integration tests (requires LLM access)
uv run pytest --agent

# Run linters and formatters
uv run prek run --all-files

# Activate virtual environment (to skip 'uv run' prefix)
source .venv/bin/activate

# Run Chainlit development server
uv run chainlit run chainlit/app.py -w
```

### Frontend Commands
```bash
# Development server with hot reload
npm run dev          # http://localhost:5173

# Production build
npm run build        # Output to frontend//

# Preview production build
npm run preview

# Run linter
npm run lint
```

### Running Backend Tests
```bash
# Run a single test file
uv run pytest tests/test_agent.py

# Run tests matching a pattern
uv run pytest -k "test_name_pattern"

# Run with verbose output
uv run pytest -v

# Run specific test markers
uv run pytest -m agent  # Same as --integration flag
```

## Architecture

### Frontend Architecture

The React frontend provides a modern web interface with:

- **Three-column layout**: Logo/avatar sidebar, collapsible prompt builder, main content area
- **Component-based architecture**: Modular React components with TypeScript
- **CSS Modules**: Scoped styling with design tokens
- **React 19 features**: React Compiler for automatic optimization
- **Vite bundler**: Fast development and optimized production builds

See [frontend/CLAUDE.md](./frontend/CLAUDE.md) for detailed frontend documentation.

### Backend Architecture (LangGraph Agent)

The backend uses LangGraph's `create_react_agent` with a custom state schema and two primary tools:

1. **select_dataset** - Searches ChromaDB vector store (Mistral embeddings) to find matching climate datasets from `data/datasets.json`
2. **create_chart** - Generates SQL queries (via Codestral) to extract data from S3 parquet files, then creates Plotly visualizations

**Agent Flow:**
```
User Query → Agent (Mistral chat model) → Tool Selection
                                        ↓
                            select_dataset (ChromaDB similarity search)
                                        ↓
                            create_chart (Codestral generates SQL → DuckDB execution → Codestral generates Plotly code)
                                        ↓
                            Visualization + Response
```

### State Management

Backend state via custom `AgentState` (extends `AgentStatePydantic`):
- `dataset`: Selected dataset metadata from ChromaDB
- `chart_query`: Generated SQL query for data extraction
- `python_code`: Generated Plotly visualization code
- `chart_data`: Processed data for visualization
- `chart`: Final Plotly chart JSON

State is persisted via `InMemorySaver` checkpointer with thread-based conversation history.

### Code Structure

```
# Backend Structure
src/atlas_assistant/
├── agent.py           # LangGraph agent creation and system prompt
├── state.py          # AgentState schema definition
├── settings.py       # Pydantic settings with Mistral API config
└── tools/
    ├── select_dataset.py    # ChromaDB-based dataset retrieval
    └── create_chart.py      # SQL generation + Plotly visualization

# Frontend Structure
frontend/
├── src/
│   ├── components/          # React components
│   │   ├── PromptBox/      # Input component with context tags
│   │   └── Welcome/        # Main application layout
│   ├── assets/
│   │   └── icons.tsx       # Icon components
│   ├── App.tsx            # Root component
│   ├── main.tsx           # Application entry point
│   └── index.css          # Global styles and design tokens
├── package.json           # Frontend dependencies
├── vite.config.ts         # Vite configuration
├── tsconfig.json          # TypeScript configuration
└── CLAUDE.md              # Frontend-specific documentation

# Root Level

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AdaptationAtlas) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
