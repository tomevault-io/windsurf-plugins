---
trigger: always_on
description: **NOTE:** You should **always** update CLAUDE.md whenever you make changes to the codebase.
---

**NOTE:** You should **always** update CLAUDE.md whenever you make changes to the codebase.

# Hodoscope

## Project Status

Installable Python library + CLI (`hodoscope`) for analyzing AI agent trajectories. Extracts actions, summarizes with LLMs, embeds, and creates interactive visualizations. v2 uses single-file JSON output with arbitrary grouping. Uses LiteLLM for provider-agnostic LLM/embedding calls.

## Installation & CLI

```bash
pip install -e .           # Install package + CLI
pip install -e ".[dev]"    # With test dependencies
hodoscope --help           # Show all commands
hodoscope analyze          # Process sources → .hodoscope.json
hodoscope viz              # Visualize analysis JSONs
hodoscope sample           # FPS-based representative sampling
hodoscope info             # Show metadata
```

## File Structure

```
pyproject.toml                        # Package config, entry point, deps
README.md                             # Human-facing docs

hodoscope/                  # Main library package
├── __init__.py                       # Public API exports
├── config.py                         # Config dataclass, defaults, env loading
├── cli.py                            # Click-based CLI entry point (4 commands)
├── pipeline.py                       # Orchestration logic (analyze, viz, sample, info)
├── sampling.py                       # FPS-based sampling & projection utilities
├── io.py                             # JSON I/O, base85 encoding, grouping, filtering
├── core.py                           # Shared utilities (embed_text, run_parallel)
├── parsers.py                        # Trajectory parsing
├── actions.py                        # Action processing & summarization
├── visualization.py                  # Interactive visualizations
├── openhands/                        # OpenHands evaluation result integration
│   ├── __init__.py
│   └── convert_to_trajectory.py      # Convert OpenHands JSONL → universal format
├── docent/                           # Docent integration
│   ├── __init__.py
│   ├── export_transcripts.py         # Export from Docent collections
│   └── convert_to_trajectory.py      # Convert Docent → universal format
└── eval/                             # .eval file integration (Inspect AI)
    ├── __init__.py
    └── convert_to_trajectory.py      # Convert .eval → universal format

tests/                                # Test suite
├── __init__.py
├── conftest.py                       # Shared fixtures
├── test_io.py                        # I/O round-trip tests (no API)
├── test_api.py                       # Public Python API tests (no API)
├── test_analyze.py                   # End-to-end pipeline tests (needs API)
├── test_viz.py                       # Visualization tests (no API)
├── test_sampling.py                  # FPS sampling tests (no API, clustered mock data)
└── sample_evals/                     # Test data
    ├── sample1.eval                  # 10 samples, gpt-4o-mini, popularity
    └── sample2.eval                  # 10 samples, gpt-4o-mini, popularity
```

## Module Overview

### `hodoscope/config.py`
**Purpose:** Centralized configuration — all defaults and env loading.

**Constants (single source of truth):**
- `DEFAULT_SUMMARIZE_MODEL` — `"openai/gpt-5.2"`
- `DEFAULT_EMBEDDING_MODEL` — `"gemini/gemini-embedding-001"`
- `DEFAULT_EMBEDDING_TASK_TYPE` — `"RETRIEVAL_DOCUMENT"`
- `DEFAULT_EMBED_DIM` — `None`
- `DEFAULT_MAX_WORKERS` — `10`
- `DEFAULT_GROUP_BY` — `"model"`
- `DEFAULT_FPS_ALPHA` — `1.0`
- `DEFAULT_FPS_BETA` — `0.1`

**Helpers:**
- `_load_env()` — Load `.env` from project root or cwd

**Config dataclass:**
- `Config(summarize_model, embedding_model, embed_dim, max_workers, reasoning_effort, normalize_embeddings, summarize_prompt, fps_alpha, fps_beta)` — Processing configuration with hardcoded defaults. `embed_dim=None` means "respect the API return dimensionality." `summarize_prompt=None` means "use the default SWE-oriented prompt" (`DEFAULT_SUMMARIZE_PROMPT` from `actions.py`). `fps_alpha` and `fps_beta` control FPS density-weighted ranking behavior.
- `Config.from_env(**overrides)` — Resolve from `.env` + env vars, apply explicit overrides (None values skipped). This is the **only** place that does env-loading side effects. Does **not** validate API keys (LiteLLM will report errors at call time).

---

### `hodoscope/cli.py`
**Purpose:** Click-based CLI entry point (`hodoscope` command)

4 commands: `analyze`, `viz`, `sample`, `info`. The `analyze` command builds a `Config.from_env()` from CLI args and passes it to `pipeline.analyze()`. CLI options no longer use `envvar=` — env var resolution is handled entirely by `Config.from_env()`.

**Helpers:**
- `_build_filter(filter_strings)` — Parse `--filter KEY=VALUE` tuples into a callable predicate (returns `None` if empty). Checks `summary['metadata'][key]` with string and numeric comparison. All filters AND'd. Used by `viz` and `sample` commands.

---

### `hodoscope/pipeline.py`
**Purpose:** Public Python API and CLI orchestration.

**Public API (composable building blocks):**
- `load_eval(path, limit, sample, seed, save_samples)` — Load trajectories from .eval file. Returns `(trajectories, fields)`. Accepts `str | Path`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AR-FORUM/hodoscope](https://github.com/AR-FORUM/hodoscope) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
