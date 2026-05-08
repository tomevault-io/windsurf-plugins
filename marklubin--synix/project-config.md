---
trigger: always_on
description: Synix is **a build system for agent memory**. Declarative pipelines define how raw conversations become searchable, hierarchical memory with full provenance tracking. Change a config, only affected layers rebuild. Think `make` or `dbt`, but for AI agent memory.
---

# CLAUDE.md — Synix

## What Is Synix

Synix is **a build system for agent memory**. Declarative pipelines define how raw conversations become searchable, hierarchical memory with full provenance tracking. Change a config, only affected layers rebuild. Think `make` or `dbt`, but for AI agent memory.

The fundamental output: **system prompt + RAG**, built from raw conversations with full lineage tracking.

## Core Concepts

- **Artifact** — immutable, versioned build output (transcript, episode, rollup, core memory). Content-addressed via SHA256.
- **Layer** — typed Python object in the build DAG. `Source` for inputs, `Transform` subclasses for LLM steps, `SearchSurface`/`SynixSearch`/`FlatFile` for projections. Dependencies are object references via `depends_on`.
- **Pipeline** — declared in Python. `Pipeline.add(*layers)` routes Source/Transform to layers, projection declarations to the manifest.
- **Projection** — structured declaration in the manifest. Materialized at release time by adapters (`synix_search`, `flat_file`). Not built during `synix build`.
- **Release** — `synix release HEAD --to <name>` materializes projections from a snapshot into `.synix/releases/<name>/`. Produces a receipt.
- **Provenance** — every artifact traces back to its inputs via `parent_labels`. Always included in search results. No separate `provenance.json`.
- **Cache/Rebuild** — hash comparison via `SnapshotArtifactCache`: if inputs or prompt changed, rebuild. Otherwise skip.

Full entity model, storage format, and dataclass definitions: [docs/entity-model.md](docs/entity-model.md)
Pipeline Python API and examples: [docs/pipeline-api.md](docs/pipeline-api.md)

## Module Structure

```
src/synix/
├── __init__.py            # Public API: Pipeline, Source, Transform, SearchIndex, FlatFile, Artifact
├── core/
│   └── models.py          # Layer hierarchy (Source, Transform, SearchIndex, FlatFile, Pipeline)
├── sdk.py                 # SDK — programmatic access (Project, Release, search, init/open_project)
├── build/
│   ├── runner.py          # Execute pipeline — walk DAG, run transforms, cache artifacts
│   ├── plan.py            # Dry-run planner — per-artifact rebuild/cached decisions
│   ├── dag.py             # DAG resolution — build order from depends_on references
│   ├── pipeline.py        # Pipeline loader — import Python module, extract Pipeline object
│   ├── object_store.py    # ObjectStore — single content-addressed write path (.synix/objects/)
│   ├── refs.py            # RefStore — git-like refs (heads/main, runs/*, releases/*)
│   ├── snapshots.py       # BuildTransaction + commit_build_snapshot
│   ├── snapshot_view.py   # SnapshotView — ref-resolved reads from .synix/objects/
│   ├── error_classifier.py # Error classification — DLQ vs fatal, DeadLetterQueue
│   ├── fingerprint.py     # Build fingerprints — synix:transform:v2 scheme
│   ├── llm_transforms.py  # Bundled memory transforms + shared LLM helper functions
│   ├── parse_transform.py # Source parser — ChatGPT/Claude JSON → transcript artifacts
│   ├── merge_transform.py # Merge transform — Jaccard similarity grouping
│   ├── transforms.py      # Transform base + registry (string dispatch fallback)
│   ├── validators.py      # Built-in validators (PII, SemanticConflict, Citation, etc.)
│   ├── fixers.py          # Built-in fixers (SemanticEnrichment, CitationEnrichment)
│   ├── projections.py     # Projection dispatch
│   └── cassette.py        # Record/replay for LLM + embedding calls
├── transforms/
│   ├── __init__.py        # Re-export: MapSynthesis, GroupSynthesis, ReduceSynthesis, FoldSynthesis, Merge
│   └── base.py            # BaseTransform (legacy compat)
├── ext/
│   ├── __init__.py        # Re-export: bundled memory transforms + migration compatibility exports
│   ├── map_synthesis.py   # Generic 1:1 synthesis transform implementation
│   ├── group_synthesis.py # Generic N:M grouping synthesis transform implementation
│   ├── reduce_synthesis.py# Generic N:1 synthesis transform implementation
│   ├── fold_synthesis.py  # Generic sequential fold synthesis transform implementation
│   └── chunk.py           # Generic 1:N text chunking transform (no LLM)
├── validators/
│   └── __init__.py        # Re-export: MutualExclusion, RequiredField, PII, SemanticConflict, Citation
├── fixers/
│   └── __init__.py        # Re-export: SemanticEnrichment, CitationEnrichment
├── projections/
│   └── __init__.py        # Re-export: SearchIndexProjection, FlatFileProjection
├── search/
│   ├── indexer.py         # SQLite FTS5 — build, query, shadow swap
│   ├── embeddings.py      # Embedding provider — fastembed, OpenAI, cached
│   └── retriever.py       # Hybrid search — keyword + semantic + RRF fusion
├── mcp/
│   ├── __init__.py        # Package exports (main, mcp)
│   ├── __main__.py        # Entry point for `python -m synix.mcp`
│   └── server.py          # FastMCP server — 20 tools exposing full SDK surface
├── cli/                   # Click CLI commands
│   ├── main.py
│   ├── build_commands.py
│   ├── artifact_commands.py
│   └── ...

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marklubin/synix](https://github.com/marklubin/synix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
