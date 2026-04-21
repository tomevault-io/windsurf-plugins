---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

WorkFlow is a Python CLI toolkit for managing LaTeX projects and a unified Zettelkasten knowledge system for academic writing (thesis, lectures, exercises). It integrates Markdown note-taking, LaTeX rendering, TikZ diagrams, exercise management, and bibliography across multiple institutions (UCR, UFide, UCIMED).

## Commands

```bash
# Install (editable)
pip install -e .
# or with uv
uv sync

# Run all tests
pytest

# Lint (matches CI)
flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
flake8 . --count --exit-zero --max-complexity=10 --max-line-length=127 --statistics
```

## CLI Entry Points

Defined in `pyproject.toml` under `[project.scripts]`:

| Command    | Module              | Purpose                              |
|------------|---------------------|--------------------------------------|
| `workflow` | `main:cli`          | Main CLI (exercise, lectures, graph, tikz, validate) |
| `inittex`  | `itep.create:cli`   | Initialize LaTeX project structure   |
| `relink`   | `itep.links:cli`    | Recreate symlinks from config.yaml   |
| `cleta`    | `lectkit.cleta:cli` | Clean TeX auxiliary files            |

## Architecture

### Module Structure

- **`src/workflow/db/`** — Unified database module (SQLAlchemy 2.0). Owns ALL ORM models and repository interfaces. Two-base architecture: `GlobalBase` (`~/.local/share/workflow/workflow.db`) and `LocalBase` (`<project>/slipbox.db`). See ADR-0003, ADR-0004, ADR-0007.

- **`src/workflow/tikz/`** — TikZ standalone asset pipeline. Compiles `.tex` diagrams to PDF/SVG with incremental builds. See ADR-0006.

- **`src/workflow/validation/`** — Frontmatter validation. Validates YAML frontmatter in `.md` notes and commented YAML in `.tex` exercises.

- **`src/workflow/latex/`** — Shared LaTeX parsing utilities. `braces.py` (brace-counting macro extraction), `comments.py` (commented YAML), `normalize.py` (custom macro expansion for Moodle export). See ADR-0011, ADR-0012.

- **`src/workflow/exercise/`** — Exercise bank management. `parser.py` (parse .tex → domain objects), `moodle.py` (Moodle XML export), `generator.py` (placeholder file creation), `selector.py` (exercise selection by taxonomy), `exam_builder.py` (exam assembly), `cli.py` (8 CLI commands). See ADR-0009, ADR-0010, ADR-0011, ADR-0012.

- **`src/workflow/lecture/`** — Lectures integration. `scanner.py` (discover .tex files, register as notes), `note_splitter.py` (split notes at `%>` markers), `linker.py` (extract `\cite`/`\ref`/`\label`, update Link/Citation tables), `eval_builder.py` (bridge EvaluationTemplate to exercise bank), `cli.py` (4 CLI commands: scan, split, link, build-eval).

- **`src/workflow/graph/`** — Knowledge graph analysis and export. `domain.py` (GraphNode, GraphEdge, KnowledgeGraph), `collectors.py` (query global+local DBs), `analysis.py` (orphans, hubs, components, neighbors, stats), `dot_export.py` (Graphviz DOT), `tikz_export.py` (TikZ with spring layout), `clustering.py` (optional networkx communities), `cli.py` (6 CLI commands: orphans, stats, export-dot, export-tikz, clusters, neighbors).

- **`src/itep/`** — Init TeX Project (ITeP). Project scaffolding and management. Uses `workflow.db` for models. Config is `config.yaml` per project (pointer to DB record, see ADR ITEP/0003).

- **`src/latexzettel/`** — Zettelkasten engine. CLI + JSONL/NDJSON RPC server + Neovim Lua client. Uses `workflow.db.models.notes` via compatibility shim in `infra/orm.py`.

- **`src/lectkit/`** — Lecture utilities: `cleta` (cleanup), `nofi` (note splitting), `crete` (exercise generation — **deprecated**, use `workflow exercise create`).

- **`src/PRISMAreview/`** — Django 5.0 PRISMA systematic review web app. Backed by MariaDB. `shared_db/router.py` enables reading from shared `workflow.db`.

- **`src/appfunc/`** — Shared utilities: input validation (`FieldSpec` in `iofunc.py`), enum selection, menus.

- **`shared/latex/sty/`** — 18 LaTeX style files (SetCommands, PartialCommands, SetExercises, VectorPGF, SetZettelkasten, etc.). Symlinked into projects from `~/.local/share/workflow/sty/`.

- **`shared/latex/templates/`** — LaTeX templates for notes, exercises, lectures.

- **`shared/latex/cls/`** — texnote.cls and preamble files (moved from latexzettel/defaults/template/).

- **`shared/latex/pandoc/`** — Pandoc pipeline: filter.lua, template.tex, defaults.yaml, preprocess.py (moved from latexzettel/pandoc/).

### Key Patterns

- All CLIs use **Click** with groups and commands
- **SQLAlchemy 2.0** with `Mapped[]` annotations is the single ORM (ADR-0004)
- Data access goes through **repository Protocol interfaces** (`workflow.db.repos.protocols`)
- **Hybrid DB**: global for reference data, local per project (ADR-0003)
- XDG layout: config in `~/.config/workflow/`, data in `~/.local/share/workflow/` (ADR-0008)
- Exercise macros: extend existing `\question`, `\qpart`, `\pts` — never replace (ADR-0005)
- `.tex` files are **truth source** for exercise content; DB stores metadata index only (ADR-0010)
- Exercise CLI: `workflow exercise parse|list|sync|gc|export-moodle|create|create-range|build-exam`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LuisUma92) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
