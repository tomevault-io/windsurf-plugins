---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Bandjacks is a Cyber Threat Defense World Modeling system designed to:
- Ingest and process cyber threat intelligence from multiple sources
- Build a comprehensive knowledge graph of threat actors, techniques, and defenses
- Model attack flows and sequences based on MITRE ATT&CK framework
- Integrate with D3FEND ontology for defensive recommendations
- Provide simulation and prediction capabilities for threat behaviors

## Current Status

The project is in **production-ready state** with comprehensive threat intelligence capabilities implemented:
- **Report Ingestion**: Full async/sync PDF processing with advanced TTP extraction pipeline
- **Attack Flow Generation**: LLM-based sequence synthesis with probabilistic edges and co-occurrence modeling
- **Graph Modeling**: Neo4j-based knowledge graph with STIX 2.1 objects and RDF/OWL bridge
- **Frontend Interface**: Next.js 15.5 React UI with analytics dashboards and job tracking
- **Unified Review System**: Comprehensive human-in-the-loop validation interface
- **Optimized Pipeline**: Chunked processing for large documents with no timeouts
- **Analytics Engine**: Co-occurrence analysis, coverage analytics, and drift detection
- **Advanced APIs**: 30+ endpoints covering catalog, search, flows, defense, compliance, and more
- **Middleware Stack**: Authentication, rate limiting, tracing, and error handling
- **Caching Systems**: TechniqueCache and ActorCache for O(1) lookups
- **Testing Framework**: Comprehensive test suite with Jest and Testing Library

## Architecture Overview

The system consists of these main components:

1. **Ingestion & Mapping**: Parser, vector retriever, IE & linker, STIX mapper with ADM validation
2. **Knowledge Layer**: Neo4j property graph, RDF/OWL store via n10s, OpenSearch KNN vector store
3. **World Model**: Attack flow builder, D3FEND overlay, simulation/prediction, coverage analytics
4. **Feedback & Operations**: Review API/UI, active learning queue, model refresh, RBAC

Key technologies and standards:
- **FastAPI** with **uv** for Python package management and uvicorn server
- **Neo4j** with neosemantics (n10s) for RDF bridge and property graph storage
- **OpenSearch KNN** for vector embeddings and semantic search
- **Redis** for job queues and caching
- **STIX 2.1** with strict **ATT&CK Data Model (ADM)** validation
- **ATT&CK release pinning** via official `index.json` catalog
- **D3FEND** ontology integration for defensive mappings
- **LiteLLM** for multi-provider LLM integration
- **Middleware Stack**: Authentication (JWT), rate limiting, tracing, error handling
- **Frontend**: Next.js 15.5, React 19.1, TypeScript 5, Tailwind CSS 4
- **Testing**: Jest, Testing Library, pytest with comprehensive coverage
- **Dependencies**: Pydantic 2.5+, Sentence Transformers, PyTorch, pdfplumber
- Optional Node.js sidecar for ADM validation or JSON-Schema export

## Development Commands

```bash
# Project setup
uv sync                    # Install dependencies
uv run pytest             # Run tests

# Start services 
# For development (with hot reload):
# IMPORTANT: Always use 4 workers for proper async job handling and concurrency
uv run uvicorn bandjacks.services.api.main:app --workers 4 --host 0.0.0.0 --port 8000
# For development with auto-reload (single worker only):
# uv run uvicorn bandjacks.services.api.main:app --reload --host 0.0.0.0 --port 8000
cd ui && npm run dev      # Frontend (Next.js) on port 3000

# Development tasks
uv run ruff check .       # Lint code
uv run mypy .            # Type checking
uv run pytest            # Run all tests
uv run pytest tests/unit  # Run unit tests only
uv run pytest --coverage # Run tests with coverage

# Frontend development
cd ui && npm run dev      # Start frontend dev server
cd ui && npm run build    # Build for production
cd ui && npm test         # Run frontend tests
cd ui && npm run test:coverage # Run frontend tests with coverage

# Batch processing
python -m bandjacks.cli.batch_extract ./reports/  # Process multiple PDFs
python -m bandjacks.cli.batch_extract --api ./reports/  # Use async API

# Database setup (automatic on API startup)
# Manual initialization if needed:
python -m bandjacks.loaders.neo4j_ddl    # Create Neo4j constraints/indexes
python -m bandjacks.loaders.opensearch_index  # Create OpenSearch indexes
```

## Database Initialization & Data Loading

### Neo4j Schema Setup

The system automatically creates comprehensive Neo4j constraints and indexes on startup via `ensure_ddl()`:

**Constraints (44 total)** - Ensuring uniqueness for:
- Core STIX nodes: `AttackPattern`, `IntrusionSet`, `Software`, `Mitigation`, `Tactic`
- Operational nodes: `AttackEpisode`, `AttackAction`, `AttackFlow`
- Detection nodes: `DetectionStrategy`, `Analytic`, `LogSource`, `SigmaRule`
- Defense nodes: `D3fendTechnique`, `DigitalArtifact`
- Review/ML nodes: `CandidateAttackPattern`, `ReviewProvenance`, `JudgeVerdict`

**Indexes (100+)** - Performance optimization for:
- Name and ID lookups across all node types
- Temporal queries (created, modified, timestamp)
- Status fields (revoked, deprecated, status)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Blevene/bandjacks](https://github.com/Blevene/bandjacks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
