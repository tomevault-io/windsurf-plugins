---
trigger: always_on
description: Meta-instructions for the Codex coding agent, peer to `CLAUDE.md`. Both
---

# AGENTS.md

Meta-instructions for the Codex coding agent, peer to `CLAUDE.md`. Both
files live at the repo root because they are agent-runtime configuration,
not project documentation — `CLAUDE.md` configures Anthropic's Claude
Code; `AGENTS.md` configures OpenAI's Codex. Do NOT move either file
under `docs/` (rule 8 below covers `.md` artifacts produced *as part of
work*; these two files configure the workers themselves).

## Hard Rules

1. Write automated tests for code changes. Prefer focused unit tests plus real Postgres integration tests when database behavior is involved.
2. Run verification before claiming work complete AND before any commit: `ruff check`, `mypy src/`, and `pytest` unless the user explicitly narrows the scope or the environment blocks a command. All three must pass; do not commit on a red gate.
3. Never commit or push without explicit user permission.
4. For multi-file or high-risk implementation work, produce a written plan first and get approval before editing.
5. Before referencing existing modules, functions, schemas, fields, or command behavior, read the actual source first.
6. Bug fixes require regression tests that reproduce the original failure.
7. Do not revert unrelated user changes. Work with the current dirty tree.
8. Plans, specs, and new Markdown docs created during work belong under `docs/`: specs in `docs/specs/YYYY-MM-DD-<topic>-design.md`, plans in `docs/plans/YYYY-MM-DD-<topic>.md`.
9. No real personal data in checked-in code, tests, fixtures, docs, comments, commit messages, or PR descriptions. The repo has been explicitly PII-scrubbed — use synthetic / redacted values for names, email addresses, phone numbers, postal addresses, meeting attendees, calendar specifics, company names, customer references, and internal codenames. If a test seems to need real data, the test is wrong: parameterize the fixture or refactor the production code so it's testable with synthetic values.

## Codex Agenting

Codex does not provide Claude Code's native agent-teams (and Claude's old `TeamCreate`/`TeamDelete` tools no longer exist anywhere). The closest equivalent in Codex is sub-agents:

- Use `explorer` agents for bounded codebase questions.
- Use `worker` agents for scoped implementation work with clear file ownership.
- Use parallel agents only when the user explicitly asks for agentic/parallel/delegated work or when future session policy allows it.
- Do not describe Microsoft Teams integration as team-agent orchestration; it is unrelated.

The `superpowers` plugin is enabled for Codex when available. Prefer its workflow skills for planning, TDD, debugging, code review, verification, and subagent-driven development. If the plugin is unavailable in a session, follow the equivalent manual workflow in this file.

## Workflow

1. Plan: identify affected files, tests, risks, and verification commands.
2. Approve: wait for user approval before multi-file edits.
3. Implement: keep changes scoped and follow existing project patterns.
4. Verify: run `ruff check`, `mypy src/`, and `pytest`; for CLI-facing changes, also run a relevant `brain ...` command.
5. Review: inspect the diff for regressions, dead code, missing tests, and doc drift before final response.

## Project Overview

Second Brain is a local personal knowledge base with hybrid search, designed to be queried from coding-agent sessions. It stores selected documents, transcripts, Slack/Gmail-derived text, and notes in Postgres + pgvector, then searches with Reciprocal Rank Fusion over full-text rank and vector cosine similarity.

## Tech Stack

- Python 3.11+
- Typer CLI
- PostgreSQL 16 + pgvector in Docker on port `55432`
- Pluggable embedders via `BRAIN_EMBEDDER`: `arctic` default, `voyage`, or `qwen3`
- Extraction: `pypdf`, `pdfplumber`, `python-docx`, `markdown-it-py`
- Graph retrieval (experimental): Apache AGE (openCypher graph in-Postgres) + `networkx` (Louvain community detection) for entity-centric GraphRAG alongside the vector/FTS search; default-OFF via `BRAIN_GRAPH_ENABLED`; needs the custom AGE Postgres image
- Tests: `pytest` with real Postgres fixtures and fake embedders
- Lint/type: `ruff`, `mypy`

## Common Commands

```bash
# Setup
cp .env.example .env
python3.11 -m venv .venv
source .venv/bin/activate
pip install -e ".[dev]"
docker compose up -d
brain init
brain reembed
brain doctor

# Daily use
brain ingest <file>
brain ingest-dir <dir>
brain search "query"
brain show <id-prefix>
brain status

# GraphRAG (experimental — entity graph alongside vector/FTS search).
# Requires the custom Apache AGE image (second-brain-age:pg16-v1.5.0-rc0-pgv0.8.2),
# NOT the stock pgvector prod image. `pip install -e ".[dev]"` pulls networkx.
# Set BRAIN_GRAPH_ENABLED=true in .env to enable the ingest-time graph sync.
brain init                               # also bootstraps AGE + graph migrations on an AGE image
brain graphrag build --backfill          # backfill the people graph from existing docs
brain graphrag communities build         # detect + summarize communities (needed for --mode global)
brain graphrag search "query"            # graph retrieval (modes: auto|local|themes|global|fuse)
brain graphrag themes --person "Jane Doe"  # "themes in my conversations with X"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mshtawythug/second-brain](https://github.com/mshtawythug/second-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
