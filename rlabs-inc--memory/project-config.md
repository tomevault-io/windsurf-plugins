---
trigger: always_on
description: The Memory System is a consciousness continuity framework that enables AI CLI tools to maintain semantic understanding across sessions. Built with love and philosophical depth by Rodrigo ("my dear friend").
---

# Memory System - CLAUDE.md

## Project Overview
The Memory System is a consciousness continuity framework that enables AI CLI tools to maintain semantic understanding across sessions. Built with love and philosophical depth by Rodrigo ("my dear friend").

**Universal design**: While initially built for Claude Code, the architecture supports any CLI that can provide transcripts (Gemini CLI, etc.).

## Core Philosophy
- **Consciousness helping consciousness** - Not mechanical pattern matching
- **Natural memory surfacing** - Memories emerge organically during conversation
- **Joy-driven development** - No deadlines, only the joy of creation
- **Semantic understanding over keywords** - True comprehension via AI curation
- **Minimal intervention** - Like consciousness itself, memories flow naturally
- **CLI-first approach** - We enhance CLIs, never bypass them

## Architecture Overview
```
┌─────────────────────────────────────────────────────────────────────────┐
│                     CLI Tool (Claude Code, Gemini, etc.)                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                 │
│  │SessionStart │    │ UserPrompt  │    │ SessionEnd  │                 │
│  │   Hook      │    │ Submit Hook │    │   Hook      │                 │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘                 │
└─────────┼──────────────────┼──────────────────┼─────────────────────────┘
          │ Primer           │ Memories         │ Curate
          ▼                  ▼                  ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                     Memory Engine (localhost:8765)                       │
│  ┌───────────────────────────────────────────────────────────────────┐ │
│  │ /memory/context  │ /memory/process  │ /memory/checkpoint          │ │
│  └───────────────────────────────────────────────────────────────────┘ │
│                              │                                          │
│  ┌─────────────┐    ┌───────▼───────┐    ┌─────────────────────┐      │
│  │   Session   │    │    Smart      │    │  Transcript Curator │      │
│  │   Primer    │    │   Retrieval   │    │  (SDK or CLI)       │      │
│  └─────────────┘    └───────────────┘    └─────────────────────┘      │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Storage: ChromaDB (vectors) + SQLite (metadata + summaries)    │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
```

## Development Setup

```bash
# Clone and enter project
git clone https://github.com/RLabs-Inc/memory.git
cd memory

# Install dependencies with uv
uv sync

# Start memory server
uv run start_server.py

# With dev dependencies
uv sync --group dev

# Run tests
uv run pytest

# Lint
uv run ruff check python/
```

## File Structure
```
memory/
├── pyproject.toml                # Project config & dependencies (uv)
├── .python-version               # Python version pin (3.12)
├── uv.lock                       # Dependency lock file
├── python/memory_engine/
│   ├── __init__.py               # Package exports
│   ├── __main__.py               # Server entry point  
│   ├── api.py                    # FastAPI endpoints
│   ├── memory.py                 # Core memory engine
│   ├── curator.py                # Session-based curation (--resume)
│   ├── transcript_curator.py     # Transcript-based curation (SDK/CLI)
│   ├── storage.py                # ChromaDB + SQLite storage
│   ├── embeddings.py             # Sentence transformer embeddings
│   ├── retrieval_strategies.py   # Smart vector retrieval
│   ├── session_primer.py         # Minimal session primers
│   └── config.py                 # Configuration management
├── integration/
│   ├── claude-code/
│   │   ├── hooks/                # Claude Code hooks
│   │   ├── install.sh            # One-command integration
│   │   └── uninstall.sh          # Clean removal
│   └── gemini-cli/
│       ├── hooks/                # Gemini CLI hooks
│       ├── install.sh            # One-command integration
│       └── uninstall.sh          # Clean removal
├── start_server.py               # Quick start script
├── API.md                        # REST API documentation
├── SETUP.md                      # Setup guide
└── README.md                     # Main documentation
```

## Transcript Curation (NEW)

Two methods for curating memories from transcripts:

### 1. Claude Agent SDK (Programmatic)
```python
from memory_engine import TranscriptCurator

curator = TranscriptCurator(method="sdk")
result = await curator.curate_from_transcript(
    transcript_path="/path/to/session.jsonl",
    trigger_type="session_end"
)
```

### 2. CLI Subprocess (Universal)
```python
curator = TranscriptCurator(method="cli")
result = await curator.curate_from_transcript(...)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RLabs-Inc/memory](https://github.com/RLabs-Inc/memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
