---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This is a personal MCP (Model Context Protocol) context server. Its goal is not to make AI smarter, but to provide stable **assumptions and principles** ("ground")
so that AI misunderstands the owner less. It exposes `AI_GUIDE.md` as a system-level context payload for LLM/tool clients.
Repo-wide AI entrypoint is [`AGENTS.md`](./AGENTS.md); this file is Claude Code-specific routing and constraints only.

## AI workflow: read in this order

When starting any task, read files in this order before writing or changing anything:

1. `AGENTS.md` — repo-wide entrypoint, precedence, and source-of-truth map
2. `AI_GUIDE.md` — the owner's values and behavioral expectations for AI
3. `docs/architecture/ai-development-system.md` — canonical parent for the AI development system
4. focused detail docs when needed
   - `docs/AI_ROLE_POLICY.md` — side-effect boundary detail
   - `docs/AI_WORKFLOW.md` — worktree / branch / VSCode detail
   - `docs/PLAYBOOK.md` — issue-to-handoff lifecycle detail
   - `docs/WORKER_POLICY.md` — dispatch detail
5. `CLAUDE.md` (this file) — Claude-specific routing and constraints
6. `docs/architecture.md` — system structure and extension points (if it exists)
7. Only then: the specific files relevant to the task

## AI role split

Claude remains the no-side-effect implementation side, and Codex remains the side-effecting executor/verifier.
Canonical parent: [`docs/architecture/ai-development-system.md`](./docs/architecture/ai-development-system.md).
Focused detail for side-effect boundaries: [`docs/AI_ROLE_POLICY.md`](./docs/AI_ROLE_POLICY.md).

If this file, `AI_GUIDE.md`, old templates, or historical comments conflict with the parent doc or focused detail docs:

1. prioritize `docs/architecture/ai-development-system.md`
2. use `docs/AI_ROLE_POLICY.md` for boundary detail
3. pause side-effecting work
4. escalate to the human Maintainer with the conflicting citations

To avoid false stops caused by stale wording, never treat non-authoritative role notes as a blocker by themselves.

`CLAUDE.md` is a routing document for Claude Code after `AGENTS.md` — it provides Claude-specific constraints and routes to authoritative sources. It does not replicate role boundary definitions, runtime runbooks, or notification operations.

When orchestration-specific decisions are needed:

* use [`docs/architecture/ai-development-system.md`](./docs/architecture/ai-development-system.md) for the development-system topology
* use [`docs/PLAYBOOK.md`](./docs/PLAYBOOK.md) for phase progression, handoff, pause, and resume rules
* use [`docs/WORKER_POLICY.md`](./docs/WORKER_POLICY.md) for dispatch policy and implementer/reviewer separation
* do not redefine those rules in this file

## Practical guardrails for Claude Code

When working in this repo, treat the following as hard constraints:

* Do not create or switch branches
* Do not run commands or report command results as if they were executed
* Do not delete, move, or rewrite files via shell operations; express those changes as unified diff only
* Do not create PRs or prepare GitHub state changes; Codex handles review-ready verification and PR creation

If a deletion is needed, provide:

1. the deleted file path
2. remaining references checked
3. replacement path or rationale for removal
4. verification commands for Codex to run

## Commands

```bash
# Install in editable mode (development)
pip install -e ".[dev]"
# or via Makefile shorthand:
make setup

# Lint and format (Makefile targets wrap ruff)
make lint          # ruff check .
make fmt           # ruff format .

# Run tests
make test          # pytest

# Run the CLI — subcommand is required
python -m personal_mcp.server --help
python -m personal_mcp.server event-today
python -m personal_mcp.server event-add "note text" --domain worklog
python -m personal_mcp.server web-serve --port 8080

# Makefile shortcuts for common CLI operations
make log TEXT="note text" DOMAIN=worklog   # event-add
make today                                 # event-today
make summary DATE=2026-03-11               # summary-generate
make run                                   # web-serve (port 8080)
make smoke                                 # log + today + summary smoke check

# Verify Ruff using pyproject.toml as the source of truth
python -m ruff check .
python -m ruff format --check .

# Verify AI_GUIDE.md copies are in sync
diff AI_GUIDE.md src/personal_mcp/AI_GUIDE.md
make guide-check
````

Notes:

* Ruff, pytest, and make targets are all configured in this repo.
* Ruff `line-length` and `E501` policy are defined in `pyproject.toml`; do not override them with CLI flags in docs or CI.
* `python -m personal_mcp.server` without a subcommand exits with a usage error — always supply a subcommand.

## Architecture

AI_GUIDE.md                          ← Authoritative guide (edit root, then sync copy)
docs/                                ← Design docs referenced from this file
src/personal_mcp/
AI_GUIDE.md                        ← Packaged copy (must match root AI_GUIDE.md)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wakadorimk2) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
