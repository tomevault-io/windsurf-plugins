---
trigger: always_on
description: This file contains detailed guidance for coding agents working in this repository.
---

# CODING AGENTS

This file contains detailed guidance for coding agents working in this repository.

---

## Context Specification Document

*Generated: 2026-03-29 | Updated: 2026-04-04 — bookmark-check CLI, batch import (--file), logging module, html.parser, HTTP retry, integration tests*

---

### Quick navigation (table of contents)

- [Read this first (progressive disclosure)](#read-this-first-progressive-disclosure)
- [1. System Purpose](#1-system-purpose)
  - [1.1 Development Commands](#11-development-commands)
- [2. Entry Points & Execution Flow](#2-entry-points--execution-flow)
- [3. Core Modules & Responsibilities](#3-core-modules--responsibilities)
- [4. Data Models & Schemas](#4-data-models--schemas)
- [5. External Dependencies & I/O](#5-external-dependencies--io)
- [6. Known Pain Points / Technical Debt](#6-known-pain-points--technical-debt)

### Read this first (progressive disclosure)

If you only need to make a small, safe change, read these sections in order and stop early when you have enough context:

1. **Section 1.1** — Development commands (`uv run ...`)
2. **Section 2** — Entry points + high-level execution flow
3. **Section 3** — Module table (open only the files you are changing)
4. **Sections 4–6** — Data models, I/O, and technical debt (deep dive only when needed)

Summary behavior at a glance:

1. `summarize` CLI
2. classifier-provided summary from `call_llm(...)`
3. direct LLM fallback (`summarize_with_llm(...)`)
4. heuristic fallback (`infer_summary(...)`)

Common edit recipes:

- **Tune folder/type/tags decisions:** start in `bookmark_tools/classify.py`.
- **Tune summary behavior:** start in `bookmark_tools/summarize.py`.
- **Tune CLI flow / dry-run output:** start in `bookmark_tools/cli.py`.
- **Verify quickly without writing files:** `uv run bookmark <URL> --dry-run`.

Files not to Commit:

- PLAN.md or other planning related files

---

### 1. System Purpose

**One-sentence description:** A CLI tool that fetches a web page by URL and creates a structured bookmark note in an Obsidian vault.

Classification is LLM-first (with heuristic fallback), and summary generation is `summarize`-first with classifier/LLM/heuristic fallbacks.

**Primary technology stack:**
- Python 3.12 (stdlib only — no third-party packages required at runtime)
- [uv](https://docs.astral.sh/uv/) for project and dependency management
- `urllib.request` for HTTP fetching (page content + LLM API)
- `html.parser` for robust HTML metadata extraction (title, meta tags, lang attribute)
- `subprocess` + `shutil.which` for optional `summarize` CLI invocation
- `re` + `html` for HTML cleaning (no BeautifulSoup/lxml)
- `json` for LLM request/response serialization
- `dataclasses` + `typing.TypedDict` for data models
- `logging` for structured diagnostic output across all modules
- Optional: any OpenAI-compatible chat completions API (`gpt-4.1-mini` default model; OpenAI/OpenRouter compatible config)
- Optional: external `summarize` CLI (`summarize --help`) for primary summary generation

### 1.1 Development Commands

Use `uv run` to execute tools inside the project environment:

```bash
# Run tests
uv run pytest tests/

# Lint Python code
uv run ruff check bookmark_tools tests

# Auto-format Python code
uv run ruff format bookmark_tools tests

# Verify formatting without writing changes
uv run ruff format --check bookmark_tools tests
```

---

### 2. Entry Points & Execution Flow

#### Primary entry point — bookmark creation (via uv)
```
uv run bookmark <URL> [--dry-run] [--disallow-new-subfolder]
```
Uses the `bookmark` script entry point defined in `pyproject.toml`, which calls `bookmark_tools:main`.

#### Alternative entry point — bookmark creation
```
uv run python -m bookmark_tools <URL> [--dry-run] [--disallow-new-subfolder]
```
Invokes `bookmark_tools/__main__.py` → calls `cli.main()`.

| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| `<URL>` | Yes | — | Web page URL to fetch and classify |
| `--file`, `-f` | No | None | Read URLs from a file (one per line); use `-` for stdin |
| `--dry-run` | No | False | Print target path + rendered note and exit; **does not write files** |
| `--disallow-new-subfolder` | No | False | Restrict classifier output to existing folders only |
| `--verbose`, `-v` | No | False | Enable verbose (debug) logging output |
| `--quiet`, `-q` | No | False | Suppress all logging output except errors |

#### Bookmark search entry point
```
uv run bookmark-search <QUERY> [--folder <FOLDER>] [--tag <TAG>] [--limit <N>] [--rebuild]
uv run bookmark-search <QUERY> --semantic [--threshold <FLOAT>] [--tag <TAG>] [--limit <N>]
uv run bookmark-search <QUERY> --hybrid [--threshold <FLOAT>] [--tag <TAG>] [--limit <N>]
uv run bookmark-search <QUERY> --show-chunks
```
Uses the `bookmark-search` script entry point defined in `pyproject.toml`, which calls `bookmark_tools.search:main`.

| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| `<QUERY>` | Yes | — | Search query text |
| `--folder` | No | None | Restrict results to a folder and its subfolders (e.g., `ML-AI`) |
| `--tag` | No | None | Restrict results to bookmarks with the given tag |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mengkeat/bookmark-tools](https://github.com/mengkeat/bookmark-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
