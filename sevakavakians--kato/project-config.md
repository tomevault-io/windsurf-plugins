---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md - AI Assistant Navigation Index

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 📖 Documentation Navigation

**Start here**: See [docs/00-START-HERE.md](docs/00-START-HERE.md) for complete documentation index organized by role and audience.

### Quick Links by Task

- **Getting started with KATO**: [docs/users/quick-start.md](docs/users/quick-start.md)
- **Code examples**: [examples/README.md](examples/README.md) - Python client, token matching, hierarchical training
- **Understanding architecture**: [docs/developers/architecture.md](docs/developers/architecture.md) + [ARCHITECTURE_DIAGRAM.md](ARCHITECTURE_DIAGRAM.md)
- **Hybrid architecture (ClickHouse + Redis)**: [docs/developers/hybrid-architecture.md](docs/developers/hybrid-architecture.md)
- **Node isolation (kb_id)**: [docs/developers/kb-id-isolation.md](docs/developers/kb-id-isolation.md)
- **Database schema (ClickHouse/Redis/Qdrant fields)**: [docs/reference/database-schema.md](docs/reference/database-schema.md)
- **Filter pipeline tuning (MinHash/LSH)**: [docs/reference/filter-pipeline-guide.md](docs/reference/filter-pipeline-guide.md)
- **Deploying to production**: [docs/operations/docker-deployment.md](docs/operations/docker-deployment.md)
- **Understanding algorithms**: [docs/research/README.md](docs/research/README.md)
- **Integration patterns**: [docs/integration/README.md](docs/integration/README.md)
- **Release management**: [docs/maintenance/releasing.md](docs/maintenance/releasing.md)
- **API reference**: [docs/reference/api/](docs/reference/api/)
- **Configuration reference**: [docs/reference/configuration-vars.md](docs/reference/configuration-vars.md)
- **Historical documentation**: [docs/archive/](docs/archive/) - Past optimizations, evaluations, and completed phases

## Project Overview

KATO (Knowledge Abstraction for Traceable Outcomes) is a deterministic memory and prediction system for transparent, explainable AI. It processes multi-modal observations (text, vectors, emotions) and makes temporal predictions while maintaining complete transparency and traceability.

**Core Concept**: **Patterns** - learned sequences (temporal) or profiles (non-temporal) that represent knowledge.

### Storage Architecture (v3.0+)
**IMPORTANT**: KATO now uses a **ClickHouse + Redis hybrid architecture** (MongoDB completely removed as of v3.0.0):
- **ClickHouse**: Pattern data storage with multi-stage filter pipeline (billion-scale performance)
- **Redis**: Session management, pattern metadata (frequency, emotives), caching
- **Qdrant**: Vector embeddings (unchanged)
- **Node Isolation**: Via `kb_id` partitioning (ClickHouse) and key namespacing (Redis)

See [docs/developers/hybrid-architecture.md](docs/developers/hybrid-architecture.md) for complete details.

### Stateless Processor Architecture (v3.0+)
**IMPORTANT**: KATO processors use a **externally stateless architecture** with an internal bridge pattern:
- **Externally Stateless**: API contract is stateless — session state passed in/out per request
- **Bridge Pattern Internally**: Session state (STM, emotives, metadata) is temporarily loaded into processor instance variables for the duration of a request, then extracted back to session state. This is marked with `BRIDGE:` comments and `TODO (Phase 1.6/1.7)` for future refactoring.
- **Config-as-Parameter**: Configuration passed as parameters, not stored in processor state
- **True Concurrency**: No locks required, unlimited concurrent sessions per node_id (because each request loads/unloads its own state)
- **Horizontal Scalability**: Processors can be scaled horizontally

**Pattern**:
```python
# OLD (v2.x - stateful, with locks)
processor.observe(observation)  # MUTATES self.stm
predictions = processor.get_predictions()  # READS self.stm

# NEW (v3.0+ - stateless, no locks)
new_state = processor.observe(observation, session_state, config)  # PURE FUNCTION
predictions = processor.get_predictions(session_state=session_state, config=config)  # PURE FUNCTION
```

**Benefits**:
- ✅ Complete session isolation (no data leaks between sessions)
- ✅ Zero lock contention (true concurrent execution)
- ✅ Simplified debugging (no hidden state)
- ✅ Deterministic behavior (same inputs → same outputs)
- ✅ Horizontal scaling (stateless processors can be replicated)

## Essential Development Commands

### Dependency Management
```bash
# After editing requirements.txt, regenerate lock file
pip-compile --output-file=requirements.lock requirements.txt
docker compose build --no-cache kato
```

### Building and Running
```bash
./start.sh                    # Start all services
docker compose down           # Stop services
docker compose restart        # Restart services
docker compose ps             # Check status
docker compose logs kato      # View logs
```

### Service URLs
- **KATO**: http://localhost:8000
- **API Docs**: http://localhost:8000/docs
- **ClickHouse**: http://localhost:8123
- **Qdrant**: http://localhost:6333
- **Redis**: redis://localhost:6379

### Testing
```bash
./start.sh  # Services must be running first!

# Run all tests
./run_tests.sh --no-start --no-stop

# Run specific test suites

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sevakavakians/kato](https://github.com/sevakavakians/kato) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
