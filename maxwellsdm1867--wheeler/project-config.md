---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this codebase is

Wheeler is a Python package that turns Claude Code into a provenance-tracked research assistant. It is not an agent framework: there is no orchestration layer. Claude Code is the orchestrator; Wheeler provides (a) MCP tools that mutate a Neo4j knowledge graph, and (b) `/wh:*` slash commands that act as mode-restricted system prompts. Everything runs locally on a Max subscription via `claude -p` subprocess. No API keys are used, ever.

Version is `0.9.15`. 1929 tests, 50 MCP tools across 5 servers.

## Commands

```bash
# Environment (Python 3.11+).  uv is the primary workflow; the pip path
# remains supported for contributors without uv installed.
uv sync --extra dev            # creates .venv/, installs from uv.lock
# or:
source .venv/bin/activate
pip install -e ".[test]"       # core + test deps (fastembed + numpy are in core)
pip install -e ".[dev]"        # full toolchain: tests + ruff + mypy + build
pip install -e ".[search]"     # legacy no-op (fastembed+numpy already in core)

# Tests
python -m pytest tests/ -q                                   # full suite, quiet
python -m pytest tests/test_merge.py -v                      # one file
python -m pytest tests/test_merge.py::TestExecuteMerge -v    # one class
python -m pytest tests/ -k "consistency"                     # by keyword
python -m pytest tests/e2e/ -v                               # e2e, needs running Neo4j
python -m pytest tests/e2e/test_graph_first_acts.py -v       # graph-first act workflows
python -m pytest tests/e2e/test_plan_lifecycle.py -v         # plan lifecycle + triple-write

# E2E test philosophy: simulate the actual act workflow against live Neo4j.
# Each test walks through the exact tool-call sequence an act prescribes:
#   1. WRITE to graph (add_plan, ensure_artifact, add_finding, link_nodes)
#   2. READ back via the same query the act uses (query_plans, query_notes)
#   3. Verify graph state with direct Cypher (status, relationships, properties)
#   4. Verify triple-write artifacts on disk (knowledge/*.json, synthesis/*.md)
# This catches drift between what the act prompt says and what the graph
# actually stores. If an act says "call query_plans(status=approved)" but
# the query returns nothing because the status enum changed, the e2e test
# fails. Unit tests with FakeBackend can't catch that.

# Lint + type check (run by pre-commit hook)
.venv/bin/ruff check wheeler/
.venv/bin/mypy wheeler/ --ignore-missing-imports

# Git hooks (strongly recommended — block commits with API key leaks, broken tests, lint errors)
wh hooks install    # copies .githooks/pre-commit + pre-push into .git/hooks/
wh hooks test       # runs pre-commit checks without committing

# Headless Claude runs (for background tasks)
wh queue "prompt"   # sonnet, 10 turns, structured JSON log to .logs/
wh quick "prompt"   # haiku, 3 turns, one-shot
wh dream            # graph consolidation (promotes tiers, detects communities, flags stale)

# MCP servers (launched by Claude Code via .mcp.json — not typically invoked by hand)
python -m wheeler.mcp_server       # legacy monolith (50 tools in one process)
python -m wheeler.mcp_core         # split: health/context/search/cypher/schema (12)
python -m wheeler.mcp_query        # split: read-only query_* (10)
python -m wheeler.mcp_mutations    # split: add_*, link, unlink, delete, merge, update (18)
python -m wheeler.mcp_ops          # split: staleness, citations, consistency, ops (10)
```

The `wheeler` and `wheeler-tools` console scripts both point at the Typer CLI (`wheeler.tools.cli:app`): `wheeler show F-3a2b`, `wheeler graph status`, `wheeler validate ...`, `wheeler install`, etc.

## Architecture (what you need to hold in your head)

### The four-layer model

```
ACTS         .claude/commands/wh/*.md      slash commands = system prompts
FILE SYSTEM  .notes/, .plans/, docs/       prose artifacts live as real files
SYNTHESIS    synthesis/*.md                Obsidian-compatible, auto-generated
GRAPH        knowledge/*.json + Neo4j      metadata + relationships only
```

The graph is an **index over files**, not a document store. A node stores `id`, `type`, `tier`, `title` (~100 chars), `path`, timestamps, and filterable metadata. Full content lives in `knowledge/{id}.json`. Human-browsable rendering lives in `synthesis/{id}.md` with YAML frontmatter and Obsidian `[[backlinks]]`. When a query needs content, it reads the JSON file; when it needs connections, it queries Neo4j.

### Triple-write (load-bearing invariant)

Every mutation (`add_finding`, `link_nodes`, `set_tier`, etc.) writes **three places** through `wheeler/tools/graph_tools/__init__.py::execute_tool()`:

1. Neo4j graph node (via `GraphBackend` ABC)
2. `knowledge/{id}.json` (atomic tmp-rename)
3. `synthesis/{id}.md` (atomic tmp-rename, Obsidian-compatible)

Plus an embedding in `.wheeler/embeddings/` if search is enabled, plus a `WriteReceipt` in `.wheeler/repair_queue.jsonl` if any layer fails, plus a `trace_id` in `.wheeler/request_log.jsonl` for correlation.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maxwellsdm1867/wheeler](https://github.com/maxwellsdm1867/wheeler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
