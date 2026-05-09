---
trigger: always_on
description: This file provides guidance to **Claude Code** (claude.ai/code) when working with code in this repository. It is not used by Cursor, Antigravity, Rovodev, or other agents — those environments use the Cicadas skill file (`SKILL.md` / `cicadas.mdc`) alone, which includes the same implementation guardrails.
---

# CLAUDE.md

This file provides guidance to **Claude Code** (claude.ai/code) when working with code in this repository. It is not used by Cursor, Antigravity, Rovodev, or other agents — those environments use the Cicadas skill file (`SKILL.md` / `cicadas.mdc`) alone, which includes the same implementation guardrails.

## Python / Environment

Tests are written in **`unittest` style** and are commonly run with `uv run pytest`. The project currently declares `pytest` and `pytest-cov` in the dev dependency group in `pyproject.toml`, and recent CLI coverage was verified with `uv run pytest`.

## Commands

**Run all tests:**
```bash
uv run pytest
```

**Run a single test file:**
```bash
uv run pytest tests/test_kickoff.py
```

**Run the context-template regression checks:**
```bash
uv run pytest tests/test_templates.py
```

**Run a single test:**
```bash
uv run pytest tests/test_kickoff.py -k test_basic_kickoff
```

**Run with coverage:**
```bash
uv run pytest --cov=src/cicadas/scripts --cov-report=term-missing
```

**Lint:**
```bash
source .venv/bin/activate && ruff check src/ tests/
```

**Format:**
```bash
source .venv/bin/activate && ruff format src/ tests/
```

**CLI scripts**:
```bash
python src/cicadas/scripts/cicadas.py init
python src/cicadas/scripts/cicadas.py status
python src/cicadas/scripts/cicadas.py check
python src/cicadas/scripts/cicadas.py kickoff {name} --intent "..."
python src/cicadas/scripts/cicadas.py kickoff {name} --intent "..." --worktree
python src/cicadas/scripts/cicadas.py branch {name} --intent "..." --modules "mod1,mod2" --initiative {name}
python src/cicadas/scripts/cicadas.py branch {name} --intent "..." --modules "mod1,mod2" --initiative {name} --worktree
python src/cicadas/scripts/cicadas.py signal "message"
python src/cicadas/scripts/cicadas.py archive {name} --type {branch|initiative}
python src/cicadas/scripts/cicadas.py update-index --branch {name} --summary "..."
python src/cicadas/scripts/cicadas.py create-lifecycle {name}  # optional: --pr-specs, --no-pr-initiatives, etc.
python src/cicadas/scripts/cicadas.py open-pr [--base branch]   # open PR from current branch (gh/glab/URL/fallback); blocks on BLOCK verdict
python src/cicadas/scripts/cicadas.py review [--initiative name]  # check review.md verdict (exit 0=PASS, 1=BLOCK, 2=not found)
python src/cicadas/scripts/cicadas.py prune {name} --type {branch|initiative}
python src/cicadas/scripts/cicadas.py abort
python src/cicadas/scripts/cicadas.py history [--output path]
python src/cicadas/scripts/cicadas.py validate-skill {slug-or-path}
python src/cicadas/scripts/cicadas.py skill-publish {slug} [--publish-dir DIR] [--symlink] [--force]
python src/cicadas/scripts/cicadas.py unarchive {name}
python src/cicadas/scripts/cicadas.py emit-event --initiative {name} --type {event.type} [--data '{json}']
python src/cicadas/scripts/cicadas.py get-events --initiative {name} [--type prefix] [--since ISO] [--last N]
python src/cicadas/scripts/cicadas.py tokens --help
python src/cicadas/scripts/cicadas.py graph build
python src/cicadas/scripts/cicadas.py graph search <query>
python src/cicadas/scripts/cicadas.py graph watch
python src/cicadas/scripts/cicadas.py graph usage --view table
```

## Architecture

Cicadas is a **spec-driven development methodology toolset** for human-AI teams. It consists of two parts:

1. **The Skill** (`src/cicadas/`) — portable CLI scripts and agent instructions that can be dropped into any project.
2. **The State** (`.cicadas/`) — filesystem-based state managed by the scripts, living in the project root.

### `src/cicadas/` Structure


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ecodan/cicadas](https://github.com/ecodan/cicadas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
