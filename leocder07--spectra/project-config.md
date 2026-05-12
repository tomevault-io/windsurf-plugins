---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is Spectra?

Spectra deploys 8 AI agents (6 parallel specialists + MetaPrompter + CritiqueAgent) to analyze entire repositories across 6 dimensions — architecture, security, quality, documentation, maintainability, performance — in under 5 minutes. Python CLI. Clean Architecture.

**Tagline:** "The full spectrum of your codebase."
**One-liner:** "8 AI agents analyze your entire repository in under 5 minutes."

---

## Development Commands

```bash
# Install dependencies
pip install -e ".[dev]"

# Run CLI
spectra analyze <repo-url>
spectra analyze <repo-url> --quick    # Skip CritiqueAgent
spectra analyze <repo-url> --format json
spectra analyze <repo-url> --audit-sink stdout|file:<path>|otlp:<url>
spectra verify spectra-report.json    # Verify Ed25519 receipt

# Run tests
pytest tests/ -v
pytest tests/entities/ -v              # Unit tests only
pytest tests/integration/ -v           # Integration tests
pytest tests/ --cov=spectra --cov-report=html

# Lint and type check
ruff check src/ tests/
ruff format src/ tests/
mypy src/

# Build and publish
python -m build
pip install dist/spectra_cli-*.whl
```

---

## Architecture — ABSOLUTE RULES (NEVER VIOLATE)

### The Dependency Rule

Source code dependencies ONLY point inward:

```
Layer 1 (entities/)         → imports NOTHING from spectra package
Layer 2 (use_cases/)        → imports ONLY from entities/
Layer 3 (adapters/)         → imports from entities/ + use_cases/
Layer 4 (infrastructure/)   → imports from all inner layers
```

**Violation = immediate rejection. No exceptions.**

### Code Standards

- Functions: ≤20 lines, ≤3 parameters, cyclomatic complexity ≤10
- No `Any` type. No `# type: ignore`.
- No `print()` in src/ — use ProgressObserver port via Rich Console
- Every entity: `frozen=True` on Pydantic models. Immutable.
- Fallible operations: `Result` dataclass pattern
- All agent outputs: validated with Pydantic model BEFORE merge
- `Literal` types for enums: `Severity = Literal["critical", "high", "medium", "low", "info"]`
- Export everything from `__init__.py` with `__all__`

### Cache Subsystem

- **`CachePort` (Layer 2) + `SqliteCacheAdapter` (Layer 4).** Single `cache.db` under `${XDG_CACHE_HOME:-~/.cache}/spectra/` in WAL mode. The use-case layer never imports `sqlite3`.
- **Three caches, one DB.** `findings_cache` (per-file, Phase 1), `full_report_cache` (per-repo+versions, Phase 2 short-circuits Stages 3-5), `findings_batches` (per-`focus_area` batch, Phase 3 — the killer feature). Plus `hit_log` for telemetry.
- **Composite-key invalidation, no policy.** Every key bundles `(content, dimension, model, prompt, schema, spectra)`. A stale row never matches a current-context lookup; physical deletion is deferred to `spectra cache prune` (Phase 4 — shipped in PR #19).
- **`bind_run_context` once at composition root.** Atomic four-tuple binding (`model_versions, prompt_versions, schema_version, spectra_version`) — eliminates the half-bound state.
- **Telemetry:** `record_hit` writes to `hit_log` per lookup; `ProgressObserver.on_cache_lookup(dim, hits, total)` surfaces the per-dimension tally in the terminal. `CacheStats.hit_rate_last_100` is the rolling rate.
- **Failure mode:** SPEC-010 — cache I/O errors degrade to no-cache for the rest of the run. **Cache failures are never fatal.**

---

## 8 Agents (6 Parallel Specialists)

```
Stage 1: INGEST     → GitPort.prepare_workspace (clone HTTPS or validate local path)
Stage 2: PLAN       → MetaPrompter (Opus 4.7 effort=medium, file tree ONLY ≤5K tokens, NEVER full code)
Stage 2½: CACHE     → Phase 2 — get_full_report(RepoCacheKey); HIT short-circuits Stages 3-5
Stage 3: ANALYZE    → Phase 3 — partition_by_cache splits batches into cached + fresh
                       Run only fresh BatchPrompts in PARALLEL via asyncio.gather:
                       Architecture + Security + Quality + Documentation + Dependency + Performance
                       (all Opus 4.7, effort=xhigh)
                       put_batch_findings(BatchCacheKey) on each success
Stage 4: MERGE      → Deduplicate findings (cached UNION fresh), cross-reference, compute scores
Stage 5: CRITIQUE   → CritiqueAgent (Opus 4.7, ADAPTIVE THINKING + task_budget, validates ALL findings)
Stage 6: REPORT     → put_full_report(RepoCacheKey, report) write-back, then render HTML/JSON/SARIF
```

### Agent Hard Rules

1. MetaPrompter NEVER gets full code. File tree only, ≤5K tokens.
2. Adaptive thinking: CritiqueAgent ONLY. No other agent uses it.
3. 6 specialists ALWAYS run in parallel: `await asyncio.gather(*agents, return_exceptions=True)`
4. Every agent output validated against Pydantic model BEFORE merge.
5. `asyncio.wait_for(timeout=120)` per agent.
6. If 2+ agents fail → abort with partial report (DEGRADED state).
7. All LLM calls through decorator chain: LoggingDecorator → RetryDecorator → AnthropicAdapter

---

## Agent Teams — File Ownership

| Teammate | Owns | Does NOT Touch |
|----------|------|----------------|
| architect-1 | `src/spectra/entities/*`, `src/spectra/use_cases/interfaces.py` | Everything else |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leocder07/spectra](https://github.com/leocder07/spectra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
