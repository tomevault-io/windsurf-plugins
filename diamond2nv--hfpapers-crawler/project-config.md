---
trigger: always_on
description: This file is for AI coding assistants (Hermes Agent, OpenCode, Claude Code, etc.)
---

# hfpapers-clawler — AI Agent Development Guide

This file is for AI coding assistants (Hermes Agent, OpenCode, Claude Code, etc.)
working on this project. It describes the project structure, key patterns, pitfalls, and constraints.

## Quick Navigation

```
~/Gitlab/Agentic4Sci/hfpapers-clawler/
├── hfpapers/             # Main Python package
├── hfpclawer/            # Download pipeline (OAI-PMH, Kaggle, monitor)
├── tests/                # pytest tests
├── scripts/              # Utility scripts (publish, OAI download)
├── docs/                 # English documentation
│   └── cn/               # 中文文档 (Chinese docs)
├── config.yaml           # Main config (YAML + .env override)
├── pyproject.toml        # Package config (setuptools)
├── run.sh                # One-click pipeline runner
├── AGENTS.md             # ← This file
└── .gitignore
```

## Core Architecture

### 3-Tier Storage

| Tier | Location | Purpose | Persistence |
|------|----------|---------|-------------|
| SQLite | `data/papers.db` | Primary store (3 tables) | Persistent |
| JSON | `data/candidates_latest.json` | Fast query cache | Overwrite |
| Files | `pdfs/` `mds/` | Download results | Persistent |

### Key Data Flow

```
HF CLI → arXiv verify → Keyword classify → Dedup → paper_store (SQLite)
                                                      ↓
                                              PDF download → MD convert
```

### Module Dependency Chain

```
sources.py       — Multi-source search (HF/OpenReview/PwC/arXiv)
       ↓
paper_store.py   — SQLite store (Snowflake + Crossref)
       ↓
evolved.py       — Crawl engine (HFPapersCrawler / DedupEngine / PaperDownloader)
       ↓
cli.py           — Typer CLI (10+ subcommands)
mcp_server.py    — MCP Server (7 tools)
```

### Config Loading

```python
from hfpapers.config import load_config, get

cfg = load_config()            # Load YAML + .env
val = get("search.queries")    # Dot-separated access
```

Config search order: `config.yaml` → `.env` (env only overrides API keys)

### Global Singletons

`paper_store.py` exposes high-level interfaces:

```python
from hfpapers.paper_store import get_store, get_crossref, ensure_paper, store_stats

store = get_store()          # PaperStore singleton
cr = get_crossref()          # CrossrefClient singleton
sf_id, is_new = ensure_paper(arxiv_id, title, ...)  # Write + dedup + cross-verify
stats = store_stats()        # Statistics
```

## Development Commands

```bash
source venv/bin/activate    # Must activate
ruff format .               # Format (line-length=100, double quotes)
ruff check .                # Lint
pyright .                   # Type check (0 errors)
python -m pytest tests/ -v  # Run tests
python -m build             # Build package
```

## Testing Guidelines

### Provided Fixtures

`tests/conftest.py` provides:
- `test_env` — auto-isolated temp directory + minimal config.yaml
- `paper_store` — in-memory SQLite PaperStore instance

### Test Strategy

| Category | Coverage | External Dependencies |
|----------|----------|----------------------|
| Unit | paper_store CRUD, Snowflake, config | None |
| Unit | DedupEngine, RelevanceDetector | None |
| Unit | HardwareProbe | psutil |
| Integration | paper_store ↔ SQLite | SQLite |
| Integration | sources search | Mock |

Creating new tests:
1. `tests/test_<module>.py`
2. Use `test_env` fixture for environment isolation
3. Mock network requests (requests / subprocess)
4. Don't depend on external API responses

## Developer Conventions

### PEP8 Internationalization Standards

All Python files MUST be 100% English-only:
- **Comments** — English only (docstrings, inline comments, block comments)
- **Strings** — English only (print, log, error messages, CLI output)
- **Variable/function/class names** — English only (PEP8 naming)
- **No Chinese characters, emoji, or box-drawing characters** in .py/.yaml/.sh/.md files

Why: `conda` environment has `LC_ALL=C` which causes `UnicodeEncodeError` on non-ASCII output.

Every `.py` file must have header:
```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
```

Exceptions (Chinese allowed):
| Location | What | Reason |
|----------|------|--------|
| `docs/cn/` | Chinese documentation | Intended for Chinese readers |
| `.hermes/` | Hermes agent plans | Internal tooling, not user-facing |
| `README.md` | `简体中文` navigation link only | One-line label |
| `AGENTS.md` | `中文文档` directory reference only | One-line comment |
| `docs/CHANGELOG.md` | Changelog entries | English only (PEP8 compliance) |

### Chinese Documentation Convention

- Chinese docs live in `docs/cn/*.zh-CN.md`
- Must be **line-to-line translations** of English originals (same line count)
- This enables: diff tracking, side-by-side editing, automated sync checks
- Update English first, then mirror edits to Chinese version

### PyPI Package Release Checklist

Before tagging a release:

```bash
# 1. Format & lint
ruff format .
ruff check --fix .

# 2. Type check
pyright .

# 3. Test
python -m pytest tests/ -v

# 4. Verify version alignment
grep __version__ hfpapers/__init__.py  # e.g. '0.3.1'
grep ^version pyproject.toml           # Must match

# 5. Build + verify
python -m build
twine check dist/*

# 6. Tag
git tag v0.3.1
git push --tags

# 7. Publish

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [diamond2nv/hfpapers-crawler](https://github.com/diamond2nv/hfpapers-crawler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
