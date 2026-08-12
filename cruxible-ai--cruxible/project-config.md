---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

**GitHub:** https://github.com/cruxible-ai/cruxible

Cruxible Core is a deterministic decision engine with receipts. AI agents (Codex, etc.) write configs and orchestrate workflows. Core executes deterministically with proof — no LLM inside.

Four primitives: **Config**, **Ingest**, **Query**, **Feedback**.

## Commands

```bash
# Install dependencies
uv sync --all-extras

# Run tests
uv run pytest

# Run Docker image tests (requires Docker)
CRUXIBLE_RUN_DOCKER_TESTS=1 uv run pytest tests/test_image -m docker

# Run single test file
uv run pytest tests/test_config/test_schema.py -v

# Lint
uv run ruff check src tests

# Format
uv run ruff format src tests

# Type check
uv run mypy src
```

## Git Conventions

- Do NOT include `Co-Authored-By` lines in commit messages.
- When implementing multi-fix plans, commit each logical fix as it's completed (source + tests together). Don't defer all commits to the end — partial staging across shared files is error-prone. After all commits, prepare a review guide covering the full set.

## Review Request Conventions

- For code-change ReviewRequests in the agent-operation kit, include structured
  `change_repo`, `change_base`, and `change_head` fields. `change_head` is the
  exact reviewed commit SHA; reviewers and merge tooling should not infer it
  from the branch tip.
- Keep `ReviewRequest.summary` implementer-owned: scope, verification evidence,
  known failures, and review context. Reviewers should put requested changes and
  approval notes in `ReviewRequest.review_notes`.

## Versioning

Version lives in two places — keep them in sync:
- `pyproject.toml` (`version = "X.Y.Z"`)
- `src/cruxible_core/__init__.py` (`__version__ = "X.Y.Z"`)

The MCP server name includes the version (`cruxible-core v0.2.0`) so agents and users can confirm which build is running.

**When to bump:**
- **Patch (0.2.x):** Bug fixes, doc/prompt wording changes, test additions
- **Minor (0.x.0):** New features (tools, evaluate checks, config capabilities), breaking prompt changes
- **Major (x.0.0):** Breaking API changes (tool signatures, config schema, storage format)

**Release process:**
1. Bump version in both files
2. Rebuild kit bundles + manifest: `uv run python scripts/build_kit_bundles.py` (if a kit's config/providers changed, first `uv run cruxible lock --kit-dir kits/<kit>`); verify with `uv run python scripts/check_kit_lockfiles.py` and commit the regenerated manifest/locks
3. Commit: `Bump to vX.Y.Z`
4. Tag: `git tag vX.Y.Z`
5. Push: `git push && git push --tags`; the tag workflow publishes both PyPI packages, rebuilds bundles from the tagged source, verifies their digests against the committed manifest, and creates or updates the GitHub release assets
6. Confirm CI's `scripts/check_kit_release_assets.py` gate is green; before the tag exists it reports a notice and skips, while an existing tag requires every manifest asset URL to return HTTP 200

## Architecture

### Three Surface Layers, One Service Core

All interfaces delegate to the **service layer** (`service/`). Never duplicate orchestration logic in handlers.

```
MCP (mcp/)  ──┐
CLI (cli/)  ──┼──▶  Service Layer (service/)  ──▶  Core Modules
HTTP (server/) ┘
```

- **MCP** (`mcp/`) — Primary interface for AI agents via FastMCP. Handlers in `handlers.py` support dual-mode: library-mode (direct calls) or server-mode (delegates to `CruxibleClient`).
- **CLI** (`cli/`) — Click CLI. Commands in `commands.py` delegate to service functions.
- **HTTP** (`server/`) — FastAPI REST server with bearer-token auth. Routes in `server/routes/`. Supports HTTP and Unix Domain Socket transports.
- **Client** (`client/`) — `CruxibleClient` SDK for talking to HTTP servers. Mirrors all service operations.

### Service Layer (`service/`)

The source of truth for all business logic. Organized by concern:

- `queries.py` — Read operations (query, schema, inspect, list, stats, sample)
- `mutations.py` — Graph mutations (add_entities, add_relationships, ingest)
- `feedback.py` — Feedback collection and outcome recording
- `execution.py` — Workflow execution (plan, run, test, apply, propose, lock)
- `groups.py` — Candidate group proposal management with resolution/trust
- `analysis.py` — Constraint evaluation and candidate finding
- `snapshots.py` — State snapshots for branching/recovery
- `types.py` — All input/output types (typed dataclasses)

Service functions have consistent signatures: accept `instance: InstanceProtocol`, return typed result dataclasses.

### Instance Protocol (`instance_protocol.py`)

Structural protocols defining abstract instance/store interfaces:
- `InstanceProtocol` — Graph/config loading, snapshot creation, store access
- `ReceiptStoreProtocol`, `FeedbackStoreProtocol`, `GroupStoreProtocol`, `EntityProposalStoreProtocol`

This abstraction enables future non-SQLite backends (e.g., cloud storage) without coupling.

The concrete implementation is `CruxibleInstance` in `cli/instance.py`, which manages the `.cruxible/` directory:

```
.cruxible/
  instance.json     # Bootstrap metadata (config path, version, compatibility mirror)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cruxible-ai/cruxible](https://github.com/cruxible-ai/cruxible) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
