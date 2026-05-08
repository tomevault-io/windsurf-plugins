---
trigger: always_on
description: Generates file-level `EditPlan` from a `ChangeSpec`:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`kg_builder` extracts knowledge graphs from Python codebases using AST parsing. It identifies entities (classes, functions, variables, imports, etc.) and captures relationships between them (CONTAINS, CALLS, INHERITS, IMPORTS, INSTANTIATES, DEFINES_IN, USES). 

**Key Capabilities:**
- **KG Extraction**: Build knowledge graphs from Python code via AST parsing
- **KG Diffing**: Compare two KG JSON files to produce structured change specifications (`kg_diff.py`)
- **Edit Planning**: Generate file-level edit plans from diffs with neighbor code context (`agent_planner.py`)
- **Round-Trip Workflow**: Orchestrate full workflow from proposed KG → diff → edit plan (`round_trip.py`)
- **MCP Server**: Standalone MCP server exposing all KG tools to Claude Code, Cursor, Windsurf (`mcp_server.py`)
- **Interactive Visualizer**: React/Cytoscape.js app for exploring and editing KGs (`viz/`)

---

## Before Modifying Code - Query the Knowledge Graph First

Always use the MCP tools or agent helper functions to understand code structure before making changes.

### MCP Tools (Recommended — available in all agent sessions)

| Task | MCP Tool |
|------|----------|
| Rebuild KG after file changes | `kg_rebuild()` |
| Search for entities | `kg_find_entity(query, entity_type)` |
| Get adjacent entities | `kg_get_neighbors(entity_id, direction)` |
| Find callers of a function | `kg_get_callers(entity_id, max_depth)` |
| Extract code with context | `kg_extract_context(entity_id, max_hops)` |
| BFS traversal | `kg_traverse(start_entity_ids, max_hops)` |
| Resolve cross-file imports | `kg_resolve_import(import_entity_id)` |
| Diff two KG JSON files | `kg_diff(existing_kg_path, proposed_kg_path)` |
| Generate edit plan from diff | `kg_generate_plan(change_spec_path, existing_kg_path)` |
| Impact analysis | `kg_impact_analysis(entity_name, depth)` |
| Understand a function fully | `kg_understand_function(function_name)` |
| Export KG to JSON | `kg_export(output_path)` |

### Agent Helper Functions (for direct Python imports)

```python
from kg_builder import build_knowledge_graph, understand_function, analyze_impact

# Build a KG from a codebase
kg = build_knowledge_graph("/path/to/code", exclude_patterns=["**/tests/*"])

# Get complete context for a function
info = understand_function("parse_file")
if info["success"]:
    print(f"Found at: {info['function']['file_path']}")
    print(f"Called by: {info['called_by']}")

# Analyze impact of changing an entity
impact = analyze_impact("KGQueryEngine", depth=2)
if impact["risk_level"] == "HIGH":
    print(f"Warning: {impact['reasons']}")
```

## Commands

```bash
# Install the package in editable mode
pip install -e .

# Run all tests
pytest tests/

# Run a single test file
pytest tests/test_parser.py

# Run tests with coverage
pytest --cov=kg_builder tests/

# Build the package
python -m build

# CLI usage
kg_builder /path/to/repo --output output.json
kg_builder /path/to/file.py --verbose
kg_builder . --exclude "**/tests/*" --exclude "**/venv/*"

# KG diffing (compare two KG JSON files)
kg_builder diff existing_kg.json proposed_kg.json --output change_spec.json

# Generate edit plan from change spec
kg_builder plan change_spec.json --codebase /path/to/repo --output plan.md

# Round-trip workflow (proposed KG → diff → edit plan)
kg_builder round-trip /path/to/repo proposed_kg.json [--existing existing_kg.json]

# MCP Server (for Claude Code, Cursor, Windsurf)
python -m kg_builder.mcp_server                    # serves current directory
python -m kg_builder.mcp_server /path/to/project   # serves specific project

# Visualizer (React/Vite app)
cd viz && npm install && npm run dev    # Dev server at localhost:3000
cd viz && npm run build                 # Production build
```

## Architecture

### Core Pipeline

The main entry point `build_knowledge_graph()` in `__init__.py` runs a two-pass pipeline:
1. **Per-file pass**: For each Python file, `parse_file()` extracts entities via AST walking, then `find_all_relationships()` detects relationships from the same AST.
2. **Cross-file pass**: `SymbolResolver` builds a symbol table across all files and creates `IMPORTS_RESOLVED_TO` and `CALLS_RESOLVED` relationships linking imports/calls across file boundaries.

### Module Structure

```
kg_builder/
├── __init__.py              # Public API: build_knowledge_graph(), agent helpers
├── cli.py                   # argparse-based CLI entry point (build, diff, plan, round-trip)
├── models.py                # Entity, Relationship, KnowledgeGraph dataclasses
├── parser.py                # AST traversal for entity extraction
├── relationship_finder.py   # Relationship detection from AST
├── query_engine.py          # KGQueryEngine: graph traversal and search
├── symbol_resolver.py       # SymbolResolver: cross-file import/call resolution
├── agent_helper.py          # High-level helper functions (understand_function, analyze_impact)
├── kg_diff.py               # Knowledge graph diffing: EntityChange, RelationshipChange, ChangeSpec
├── agent_planner.py         # Edit plan generation from ChangeSpec with neighbor context

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sachdved/kg_builder](https://github.com/sachdved/kg_builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
