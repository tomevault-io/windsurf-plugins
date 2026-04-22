---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MCP Advanced Multi-Agent Ecosystem - a production-grade Model Context Protocol (MCP) server ecosystem with multiple independent MCP servers for AI-assisted development workflows.

## Build & Development Commands

### Full Setup (first-time)
```bash
./scripts/setup.sh  # Requires python3.12, node, npm
```

### Install/Rebuild All Servers
```bash
./scripts/install-mcp-servers.sh
```

### Verify Installation
```bash
./scripts/test-installation.sh
```

### Individual MCP Servers

**Context Persistence (Python)**
```bash
cd src/mcp-servers/context-persistence
pip install -e .
python3 -m context_persistence.server  # Run standalone
```

**TypeScript Servers (task-orchestrator, search-aggregator, agent-swarm, skills-manager)**
```bash
cd src/mcp-servers/<server-name>
npm install && npm run build
node dist/index.js  # Run standalone
```

**Agent Swarm (Development Mode)**
```bash
cd src/mcp-servers/agent-swarm
npm run dev   # Uses tsx for hot-reload
npm test      # Run tests
npm run lint  # Lint code
```

### Running Tests
```bash
# Agent Swarm
cd src/mcp-servers/agent-swarm
npm test                    # All tests
npm run test:watch          # Watch mode
npm run test:coverage       # With coverage

# Other TypeScript servers
cd src/mcp-servers/<server>
npm test
```

## Architecture

### MCP Server Stack

```
┌─────────────────────────────────────────────────────────────┐
│                     agent-swarm (TypeScript)                │
│  Central orchestration layer with SPARC workflows and       │
│  Boomerang task delegation. Routes to other servers.        │
└─────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌───────────────┐   ┌─────────────────┐   ┌────────────────┐
│task-orchestrator│ │search-aggregator │ │context-persistence│
│  (TypeScript) │   │   (TypeScript)  │   │    (Python)    │
│               │   │                 │   │                │
│ - Task CRUD   │   │ - Multi-source  │   │ - SQLite +     │
│ - DAG deps    │   │   search routing│   │   Qdrant       │
│ - Code exec   │   │ - Perplexity,   │   │ - Semantic     │
│ - Git linking │   │   Brave, Google │   │   search       │
│ - Analysis    │   │ - Result caching│   │ - Embeddings   │
└───────────────┘   └─────────────────┘   └────────────────┘
        │                     │                     │
        └─────────────────────┴─────────────────────┘
                              │
                      ┌───────┴───────┐
                      ▼               ▼
              ┌─────────────┐  ┌─────────────┐
              │skills-manager│ │github-oauth │
              │ (TypeScript)│  │ (Node.js)   │
              └─────────────┘  └─────────────┘
```

### Key Components

**agent-swarm** (`src/mcp-servers/agent-swarm/`)
- 7 specialized agent types: research, architect, implementation, testing, review, documentation, debugger
- SPARC workflow manager: Specification → Pseudocode → Architecture → Refinement → Completion
- Boomerang task delegation for iterative refinement
- Routes requests to appropriate downstream MCP servers

**task-orchestrator** (`src/mcp-servers/task-orchestrator/`)
- sql.js for pure JavaScript SQLite (no native compilation)
- Multi-language code execution (Python, JavaScript, Bash, SQL)
- Code quality analysis and security scanning
- DAG-based task dependency tracking
- Git commit linking

**context-persistence** (`src/mcp-servers/context-persistence/`)
- SQLAlchemy async + aiosqlite for conversations
- Qdrant local for vector embeddings
- sentence-transformers for semantic search
- Falls back to hash-based embeddings if model unavailable

**search-aggregator** (`src/mcp-servers/search-aggregator/`)
- Aggregates Perplexity, Brave, Google search providers
- Local caching with sql.js
- API key configuration via environment variables

### Storage Locations

Runtime data is stored in `~/.mcp/`:
```
~/.mcp/
├── context/
│   ├── db/          # SQLite conversation data
│   └── qdrant/      # Vector embeddings
├── tasks/           # Task orchestrator database
├── cache/
│   ├── search/      # Search result cache
│   └── code/        # Code execution temp files
├── logs/            # Server logs
├── skills/          # Skills database
└── agents/          # Agent swarm database
```

Fallback storage in `.mcp-local/` when `~/.mcp` isn't writable.

### Configuration

MCP client configuration template: `configs/roo_mcp_config.json`

Roo Code settings: `~/Library/Application Support/Code/User/globalStorage/rooveterinaryinc.roo-cline/settings/mcp_settings.json`

Environment variables for search-aggregator:
- `PERPLEXITY_API_KEY`
- `BRAVE_API_KEY`
- `GOOGLE_API_KEY` / `GOOGLE_CX`

### External MCP Servers (Git Submodules)

External MCPs are managed as **git submodules** in `src/mcp-servers/external/`:

| Server | Type | Purpose | Source |
|--------|------|---------|--------|
| **context7** | TypeScript | Documentation lookup | [Upstash](https://github.com/upstash/context7) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gr3enarr0w) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
