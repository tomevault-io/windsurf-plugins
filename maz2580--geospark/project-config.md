---
trigger: always_on
description: **GeoSpark** is an open-source Geospatial Intelligence Protocol & Engine that gives AI models genuine spatial reasoning capabilities. It is the "MCP for geospatial."
---

# GeoSpark - Project Intelligence

## Project Overview
**GeoSpark** is an open-source Geospatial Intelligence Protocol & Engine that gives AI models genuine spatial reasoning capabilities. It is the "MCP for geospatial."

- **Language**: Python 3.11+
- **Package manager**: pip (venv at `.venv/`)
- **License**: Apache 2.0
- **Status**: Phase 7A complete -- 540 tests passing, live at geospark.terrascout.app
- **PyPI**: `pip install geospark-ai[mcp]` — https://pypi.org/project/geospark-ai/
- **MCP**: `geospark-mcp` CLI command (6 tools, official MCP SDK)

## Architecture

```
geospark/
├── protocol/       # GSP schema (Pydantic models for queries/results)
├── engine/         # Spatial reasoning core (topology, CRS, distance, planner, cache)
│   ├── core.py          # Engine orchestrator + QueryChain
│   ├── spatial_reasoner.py  # Topology, distance, geometric operations
│   ├── crs_handler.py   # CRS transformations, validation
│   ├── planner.py       # Query execution planner
│   ├── temporal_engine.py   # Time-series queries, change detection
│   ├── aggregator.py    # Zonal stats, spatial joins
│   └── cache.py         # Spatial-aware LRU caching
├── rag/            # Spatial retrieval-augmented generation
│   ├── retriever.py     # Spatial + semantic feature retrieval
│   ├── chunker.py       # Context-window-sized spatial chunks
│   └── context_builder.py  # Optimal LLM context from spatial data
├── tools/          # Pluggable tools (8 registered)
│   ├── geocoding/       # Nominatim geocoder + reverse geocoder
│   ├── satellite/       # STAC client, NDVI, spectral indices
│   ├── terrain/         # Elevation (Open Elevation API)
│   ├── routing/         # OSRM route analyzer
│   ├── change_detection/  # Pixel change detection
│   └── normalized_result.py  # Consistent result shape for all tools
├── integrations/   # LLM connectors (5 providers)
│   ├── openrouter.py    # Free LLM integration (7 model aliases)
│   ├── openai_tools.py  # OpenAI function calling
│   ├── anthropic_tools.py  # Anthropic tool use
│   ├── ollama_tools.py  # Ollama (local models)
│   ├── generic.py       # Any OpenAI-compatible API
│   ├── mcp_server.py    # Monolithic MCP server (legacy)
│   └── supabase_db.py   # PostGIS spatial database backend
├── mcp_servers/    # Domain-specific MCP servers (Arion pattern)
│   ├── spatial_reasoning.py  # Topology, distance, operations
│   ├── geocoding.py     # Address <-> coordinates
│   ├── terrain.py       # Elevation queries
│   └── launcher.py      # Multi-server launcher
├── memory/         # Session persistence + spatial memory + intelligence
│   ├── session_store.py   # Save/resume conversations
│   ├── spatial_memory.py  # Persistent spatial knowledge (basic)
│   ├── intelligence.py    # Enhanced dual memory: facts + episodes + contradictions
│   └── vector_store.py    # FAISS/numpy vector storage for embedding search
├── bench/          # GeoSpark Bench evaluation framework (5 benchmarks, 535 questions)
│   ├── datasets/        # JSON benchmark datasets
│   ├── generate_datasets.py  # Dataset generator
│   ├── runner.py        # BenchRunner + load/list
│   └── scorer.py        # Scoring + parsing
├── flows/          # GeoSpark Flows (workflow automation)
│   ├── flow_schema.py   # Flow, FlowStep, FlowRoute, FlowRun models
│   ├── flow_builder.py  # Fluent builder API
│   ├── flow_runner.py   # Topological execution engine
│   └── templates.py     # Pre-built flow templates
├── knowledge/      # Spatial Knowledge Graph
│   ├── entities.py      # SpatialEntity, SpatialRelation models
│   ├── graph.py         # SpatialKnowledgeGraph (BFS, auto-relate, query)
│   └── loaders.py       # GeoJSON + Overpass loaders
├── plugins/        # Community Plugin System
│   ├── manifest.py      # PluginManifest (geospark.plugin.json schema)
│   ├── loader.py        # PluginLoader (discover, load, validate)
│   └── hooks.py         # PluginHooks (lifecycle callbacks)
├── utils/          # Shared utilities
├── api.py          # FastAPI REST server (34 endpoints, incl. /status, /memory/*)
└── cli.py          # CLI entry point (Click + Rich, 13 commands)
```

## Development Commands

```bash
# Activate venv (ALWAYS use this - never install globally)
source .venv/Scripts/activate    # Windows/Git Bash
# or: .venv\Scripts\activate     # Windows CMD

# Install in dev mode
pip install -e ".[dev]"

# Run tests
pytest tests/ -v

# Lint
ruff check geospark/ tests/
ruff format geospark/ tests/

# Type check
mypy geospark/

# CLI
python -m geospark.cli info
python -m geospark.cli geocode "Paris, France"

# Docker
docker compose up              # Full stack with PostGIS
docker compose up geospark     # Just GeoSpark API
```

## Code Style & Conventions

- **Type hints**: Required on all public functions
- **Docstrings**: Google-style on all public classes and functions
- **Models**: Pydantic v2 BaseModel for all data schemas
- **Async**: Use async where I/O is involved (httpx for HTTP)
- **Imports**: `from __future__ import annotations` at top of every module
- **Testing**: pytest with fixtures; test files mirror source structure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Maz2580) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
