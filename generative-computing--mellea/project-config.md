---
trigger: always_on
description: AGENTS.md — Instructions for AI coding assistants (Claude, Cursor, Copilot, Codex, Roo, etc.)
---

<!--
AGENTS.md — Instructions for AI coding assistants (Claude, Cursor, Copilot, Codex, Roo, etc.)
-->

# Agent Guidelines for Mellea Contributors

> **Which guide?** Modifying `mellea/`, `cli/`, or `test/` → this file. Writing code that imports Mellea → [`docs/AGENTS_TEMPLATE.md`](docs/AGENTS_TEMPLATE.md).

> **Code of Conduct**: This project adheres to a [Code of Conduct](CODE_OF_CONDUCT.md). All contributors, including AI assistants, are expected to follow these community standards when generating code, documentation, or interacting with the project.

## 1. Quick Reference

**⚠️ Always use `uv` for Python commands** — never use system Python or `pip` directly.
- Run Python scripts: `uv run python script.py` (not `python script.py`)
- Run tools: `uv run pytest`, `uv run ruff` (not `pytest`, `ruff`)
- Install deps: `uv sync` (not `pip install`)
- The virtual environment is `.venv/` — `uv run` automatically uses it

```bash
pre-commit install                    # Required: install git hooks
uv sync --all-extras --all-groups     # Install all deps (required for tests)
uv sync --extra backends --all-groups # Install just backend deps (lighter)
ollama serve                          # Start Ollama (required for most tests)
uv run pytest                         # Default: qualitative tests, skip slow tests
uv run pytest -m "not qualitative"    # Fast tests only (~2 min)
uv run pytest -m slow                 # Run only slow tests (>5 min)
uv run pytest --co -q                 # Run ALL tests including slow (bypass config)
uv run ruff format .                  # Format code
uv run ruff check .                   # Lint code
uv run mypy .                         # Type check
```
**Branches**: `feat/topic`, `fix/issue-id`, `docs/topic`

## 2. Directory Structure
| Path | Contents |
|------|----------|
| `mellea/core/` | Core abstractions: Backend, Base, Formatter, Requirement, Sampling |
| `mellea/stdlib/` | Standard library: Sessions, Components, Context |
| `mellea/backends/` | Providers: HF, OpenAI, Ollama, Watsonx, LiteLLM |
| `mellea/formatters/` | Output formatters for different types |
| `mellea/templates/` | Jinja2 templates |
| `mellea/helpers/` | Utilities, logging, model ID tables |
| `cli/` | CLI commands (`m serve`, `m alora`, `m decompose`, `m eval`) |
| `test/` | All tests (run from repo root) |
| `docs/examples/` | Example code (run as tests via pytest) |
| `.agents/skills/` | Agent skills ([agentskills.io](https://agentskills.io) standard) |
| `scratchpad/` | Experiments (git-ignored) |

## 3. Test Markers
Tests use a four-tier granularity system (`unit`, `integration`, `e2e`, `qualitative`) plus backend and resource markers. The `unit` marker is auto-applied by conftest — never write it explicitly. The `llm` marker is deprecated; use `e2e` instead.

See **[test/MARKERS_GUIDE.md](test/MARKERS_GUIDE.md)** for the full marker reference (tier definitions, backend markers, resource gates, auto-skip logic, common patterns).

**Examples in `docs/examples/`** are opt-in — unlike `test/` files (auto-collected, default `unit`), examples require an explicit `# pytest:` comment to be collected. Files without this comment are silently ignored (they won't appear in skip summaries either). This is because examples have variable dependencies and limited setup:
```python
# pytest: e2e, ollama, qualitative
"""Example description..."""
```

⚠️ Don't add `qualitative` to trivial tests — keep the fast loop fast.
⚠️ Mark tests taking >1 minute with `slow`.

## 4. Agent Skills

Skills live in `.agents/skills/` following the [agentskills.io](https://agentskills.io) open standard. Each skill is a directory with a `SKILL.md` file (YAML frontmatter + markdown instructions).

**Tool discovery:**

| Tool              | Project skills    | Global skills       | Config needed                                                      |
| ----------------- | ----------------- | ------------------- | ------------------------------------------------------------------ |
| Claude Code       | `.agents/skills/` | `~/.claude/skills/` | `"skillLocations": [".agents/skills"]` in `.claude/settings.json`  |
| IBM Bob           | `.bob/skills/`    | `~/.bob/skills/`    | Symlink: `.bob/skills` → `.agents/skills`                          |
| VS Code / Copilot | `.agents/skills/` | —                   | None (auto-discovered)                                             |

**Bob users:** create the symlink once per clone:

```bash
mkdir -p .bob && ln -s ../.agents/skills .bob/skills
```

**Available skills:** `/audit-markers`, `/skill-author`

## 5. Coding Standards
- **Types required** on all core functions
- **Docstrings are prompts** — be specific, the LLM reads them
- **Google-style docstrings** — `Args:` on the **class docstring only**; `__init__` gets a single summary sentence. Add `Attributes:` only when a stored value differs in type/behaviour from its constructor input (type transforms, computed values, class constants). See CONTRIBUTING.md for a full example.
- **Ruff** for linting/formatting
- Use `...` in `@generative` function bodies
- Prefer primitives over classes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [generative-computing/mellea](https://github.com/generative-computing/mellea) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
