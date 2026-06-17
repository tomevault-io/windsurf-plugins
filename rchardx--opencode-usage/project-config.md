---
trigger: always_on
description: Guidelines for AI agents working in this repository.
---

# AGENTS.md — opencode-usage

Guidelines for AI agents working in this repository.

## Project Overview

Python CLI tool that reads OpenCode's local SQLite database and displays token usage statistics, with an optional LLM-powered insights analysis that produces an HTML report. Uses Rich for terminal rendering, argparse for CLI with subcommands, and dataclasses for data structures.

**Two modes of operation:**

1. **`run`** (default) — Token usage tables: daily breakdown, group-by model/agent/provider/session, period comparison, JSON export.
2. **`insights`** — LLM-powered analysis pipeline: extract session transcripts → concurrent LLM facet extraction → aggregate analysis → generate a self-contained HTML report.

**Stack**: Python 3.10+, Rich, SQLite, pytest, Ruff, uv

## Repository Structure

```
src/opencode_usage/
  __init__.py        # Package init, __version__ via importlib.metadata
  __main__.py        # `python -m opencode_usage` entry
  _opencode_cli.py   # Wrapper around `opencode` binary (db path, config paths, models)
  auth.py            # Credential resolution (env → auth.json → opencode.json)
  cli.py             # Subcommand parser (run/insights), main() dispatcher
  db.py              # SQLite queries, dataclasses (TokenStats, UsageRow, OpenCodeDB)
  llm.py             # Thin OpenAI-compatible HTTP client (stdlib urllib)
  models.py          # Model discovery, tier-based ranking, interactive picker
  render.py          # Rich table rendering, formatting helpers

  insights/          # LLM-powered analysis subpackage
    __init__.py      # Re-exports: AggregatedStats, FacetCache, InsightsConfig, SessionFacet, SessionMeta
    types.py         # Dataclasses: SessionFacet, SessionMeta, AggregatedStats, InsightsConfig
    extract.py       # Session filtering, metadata extraction, transcript reconstruction, stats
    analyze.py       # Concurrent LLM facet extraction, aggregate analysis, NDJSON parsing
    cache.py         # Per-session facet cache with atomic writes (.tmp → os.rename)
    prompts.py       # Prompt builders for 8 analysis dimensions
    report.py        # HTML report generation (9-section terminal-hacker aesthetic)
    orchestrator.py  # Pipeline: extract → analyze → report, Rich progress display

tests/
  conftest.py                   # autouse fixture: clears lru_cache between tests
  test_auth.py                  # Credential resolution, list_providers
  test_cli.py                   # CLI parsing, _resolve_since, _compute_deltas, _fetch_rows
  test_db.py                    # DB queries with in-memory SQLite fixtures
  test_insights_analyze.py      # Facet extraction, aggregate analysis, NDJSON/JSON parsing
  test_insights_cache.py        # FacetCache has/get/put/clear, corruption handling
  test_insights_extract.py      # Session filtering, metadata, transcript, stats extraction
  test_insights_orchestrator.py # Pipeline orchestration, graceful degradation
  test_insights_prompts.py      # Prompt builder output validation
  test_insights_report.py       # HTML report sections, CSS, rendering helpers
  test_insights_types.py        # Dataclass defaults, field types
  test_llm.py                   # HTTP client, error handling, JSON parsing
  test_models.py                # Model listing, ranking, search, tier sorting
  test_opencode_cli.py          # CLI wrapper, path resolution, XDG fallbacks
  test_render.py                # Formatting helpers (_fmt_tokens, _spark_bar, etc.)
```

## CLI Subcommands

The CLI uses argparse subcommands. When invoked without a subcommand (or with flags only), it auto-defaults to `run`.

### `run` (default)

Token usage statistics with tabular output.

```bash
opencode-usage                         # default: last 7 days, daily breakdown
opencode-usage run --days 30
opencode-usage run --since 7d          # relative: 7d, 2w, 30d, 3h
opencode-usage run --since 2025-01-01  # ISO date
opencode-usage run --by model          # group by: model, agent, provider, session, day
opencode-usage run --by agent --limit 10
opencode-usage run --json
opencode-usage run --compare           # compare with previous period of same length
```

| Flag | Type | Default | Description |
|---|---|---|---|
| `--days N` | int | 7 | Show last N days |
| `--since SPEC` | str | — | `'7d'`, `'2w'`, `'3h'`, or ISO date |
| `--by DIM` | choice | `day` | `model`, `agent`, `provider`, `session`, `day` |
| `--limit N` | int | — | Max rows to display |
| `--json` | flag | — | Output as JSON |
| `--compare` | flag | — | Compare with previous period |

### `insights`

LLM-powered usage analysis → HTML report.

```bash
opencode-usage insights                          # interactive model picker
opencode-usage insights --model gpt-4o-mini      # specific model
opencode-usage insights --force                   # ignore cache, re-analyze
opencode-usage insights --concurrency 4           # limit parallel LLM workers
opencode-usage insights --output report.html      # custom output path
opencode-usage insights --days 30                 # last 30 days (default: 30)
```

| Flag | Type | Default | Description |
|---|---|---|---|
| `--days N` | int | 30 | Show last N days |
| `--since SPEC` | str | — | `'7d'`, `'2w'`, `'3h'`, or ISO date |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rchardx/opencode-usage](https://github.com/rchardx/opencode-usage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
