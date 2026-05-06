---
trigger: always_on
description: **What**: Python ETL pipeline streaming n8n PostgreSQL execution history → Langfuse traces via OTLP
---

# AI Coding Agent Instructions for n8n-langfuse-shipper

## Quick Reference (AI Agent Onboarding)

**What**: Python ETL pipeline streaming n8n PostgreSQL execution history → Langfuse traces via OTLP
**Why**: shipper historical workflow executions for AI observability and debugging
**Stack**: Python 3.12+, Pydantic models, OpenTelemetry SDK, psycopg3, Fish shell

### 5-Minute Orientation
- **Entry point**: `src/n8n_langfuse_shipper/__main__.py` - Typer CLI orchestrating Extract→Transform→Load pipeline
- **Core transform**: `src/n8n_langfuse_shipper/mapping/orchestrator.py` - maps n8n NodeRuns to Langfuse spans with deterministic IDs
- **Models**: `src/n8n_langfuse_shipper/models/n8n.py` + `src/n8n_langfuse_shipper/models/langfuse.py` - Pydantic validation throughout
- **Config**: `src/n8n_langfuse_shipper/config.py` - Pydantic Settings from env vars (see "Environment Variables" section)
- **Tests**: Run `pytest` or `pytest -q` in Fish shell - 45+ test files assert invariants
- **Dev workflow**: `ruff check . && mypy src && pytest` before commits (line-length cap: 100 chars)

### Critical Invariants (NEVER VIOLATE)
1. **Deterministic IDs**: UUIDv5 with `SPAN_NAMESPACE` in `mapping/id_utils.py` - changing breaks idempotency
2. **Execution ID appears once**: Root span metadata `n8n.execution.id` only (never duplicated)
3. **Parent resolution**: 5-tier precedence (Agent Hierarchy → Runtime Exact → Runtime Last → Static Graph → Root) - see "Parent Resolution" section
4. **Binary stripping**: ALWAYS unconditional (even when truncation disabled) - see "Binary & Multimodality" section
5. **Timezone aware**: All datetimes UTC-aware, NEVER use `datetime.utcnow()` (test enforces this)
6. **Fish shell**: NO Bash heredocs/arrays; use `set -x VAR value` for exports; Python one-liners: `python -c "import sys; print('ok')"` pattern

### Where to Look
- **Mapping modules**: `src/n8n_langfuse_shipper/mapping/` - 13 pure submodules (orchestrator, binary_sanitizer, generation, parent_resolution, prompt_detection, prompt_resolution, prompt_version_resolver, etc.)
- **Parent resolution rules**: "Parent Resolution" section (Precedence table) + "Agent/Tool Hierarchy" section
- **Generation detection heuristics**: "Generation Detection" section (tokenUsage presence OR provider markers)
- **Binary & Media handling**: "Binary & Multimodality" section (stripping + media token flow when `ENABLE_MEDIA_UPLOAD=true`)
- **Reserved metadata keys**: "Reserved Metadata Keys" section (root span, spans, Gemini anomaly, media flags)
- **Testing contract**: "Testing Contract" section (deterministic IDs, parent precedence, binary stripping, timezone tests)
- **Environment variables**: "Environment Variables" section (complete table with defaults)

### Common Tasks

**For development (Fish shell):**
```fish
# Run specific test file
pytest tests/test_mapper.py -v

# Type check only mapping module
mypy src/n8n_langfuse_shipper/mapping/

# Dry-run first 25 executions
n8n-shipper shipper --limit 25 --dry-run

# Check for line length violations
ruff check . --select E501
```

**For users (Bash/Zsh):**
```bash
# Run specific test file
pytest tests/test_mapper.py -v

# Type check only mapping module
mypy src/n8n_langfuse_shipper/mapping/

# Dry-run first 25 executions
n8n-shipper shipper --limit 25 --dry-run

# Check for line length violations
ruff check . --select E501
```
### Module Refactor Status
Mapper logic extracted into `src/n8n_langfuse_shipper/mapping/` subpackage (completed). Facade `src/n8n_langfuse_shipper/mapper.py` preserves public API for backward compatibility.

---

## Purpose
Python-based microservice for high-throughput shipper of historical n8n execution data from PostgreSQL to Langfuse via OpenTelemetry (OTLP) endpoint. Focus: correctness, performance, robustness for large-scale data migration.

## Document Sync & Version Policy
This file is a normative contract. Any behavioral change to mapping, identifiers, timestamps, parent resolution, truncation, binary stripping, generation detection, or environment semantics MUST be reflected here and in matching tests plus the README in the same pull request. Tests are the authority when ambiguity arises; if tests and this document diverge, update this document. Do not introduce silent behavior drift.

---

## Glossary (Authoritative Definitions)

Use these exact meanings in code comments, docs, and tests. Adding a new term? Update here in same PR.

* **Agent Hierarchy:** Parent-child relationship inferred from any non-`main` workflow connection whose type starts with `ai_` making the agent span the parent. See **Agent Parent Fixup** for timing inversion correction.
* **Agent Parent Fixup:** Post-mapping phase re-parenting tool/component spans to agent spans when tool `startTime` precedes agent `startTime` (timing inversion). Deterministic: selects latest agent span starting at or before tool; if none exist, uses earliest agent span. Emits `n8n.agent.parent_fixup=true` metadata when fixup occurs.
* **Backpressure:** Soft limiting mechanism (queue size vs `EXPORT_QUEUE_SOFT_LIMIT`) triggering exporter flush + sleep.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rwb-truelime/n8n-langfuse-shipper](https://github.com/rwb-truelime/n8n-langfuse-shipper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
