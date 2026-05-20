---
trigger: always_on
description: This file provides context for AI coding assistants (Claude Code, Cursor, GitHub Copilot, Codex, etc.) working with the Deja Vu repository.
---

# AGENTS.md

This file provides context for AI coding assistants (Claude Code, Cursor, GitHub Copilot, Codex, etc.) working with the Deja Vu repository.

## Project Overview

**Deja Vu** ("deja-vu") is an intelligent memory layer for AI agents and assistants. It provides persistent, personalized memory via both a hosted platform API and self-hosted open-source SDKs.

- **Repository**: https://github.com/dejavu-memory/dejavu
- **Documentation**: https://docs.dejavu.ai
- **License**: Apache-2.0

## Repository Structure

This is a **polyglot monorepo** containing Python and TypeScript packages, CLIs, servers, plugins, documentation, and evaluation tooling.

### Key Directories

| Directory | Description |
|-----------|-------------|
| `dejavu/` | Core Python SDK (`dejavu-memory` on PyPI) — memory, LLMs, embeddings, vector stores, graphs, rerankers |
| `dejavu-ts/` | TypeScript SDK (`dejavu-memory` on npm) — client + OSS memory |
| `cli/python/` | Python CLI (`dejavu-cli` on PyPI) — Typer-based, entry point `dejavu` |
| `cli/node/` | Node CLI (`@dejavu/cli` on npm) — Commander-based, entry point `dejavu` |
| `vercel-ai-sdk/` | `@dejavu/vercel-ai-provider` — Vercel AI SDK memory provider |
| `openclaw/` | `@dejavu/openclaw-dejavu` — OpenClaw plugin for Claude Code / AI editors |
| `server/` | FastAPI REST server for self-hosted Deja Vu (Docker: FastAPI + PostgreSQL/pgvector + Neo4j) |
| `openmemory/` | Self-hosted memory platform — `api/` (FastAPI + Alembic + MCP server) and `ui/` (Next.js 15 + React 19) |
| `dejavu-plugin/` | AI editor plugins (Claude Code, Cursor, Codex) — MCP server connection, lifecycle hooks, skills |
| `skills/` | Claude Code skill definitions. Reference skills (SDK knowledge, always-on): `dejavu/`, `dejavu-cli/`, `dejavu-vercel-ai-sdk/`. Pipeline skills (run on demand): `dejavu-integrate/`, `dejavu-test-integration/` |
| `docs/` | Documentation site (Mintlify) |
| `tests/` | Python SDK tests (pytest) |
| `evaluation/` | Benchmarking framework — LOCOMO evals, experiment runner, score generation |
| `examples/` | Sample projects — demo apps, Chrome extension, multi-agent patterns |
| `cookbooks/` | Jupyter notebooks — customer support chatbot, AutoGen integration |
| `embedchain/` | Legacy Embedchain RAG framework (maintained separately, Poetry-based) |
| `pr-reviews/` | Pull request review materials |
| `scripts/` | Repo-wide utility scripts (e.g., `check-llms-txt-coverage.py` for docs/llms.txt sync) |

### Core Package Dependencies

```
dejavu (Python SDK)          dejavu-ts (TypeScript SDK)
├── dejavu/memory/           ├── src/client/        (MemoryClient — hosted)
├── dejavu/llms/             └── src/oss/           (Memory — self-hosted)
├── dejavu/embeddings/           ├── src/llms/
├── dejavu/vector_stores/        ├── src/embeddings/
├── dejavu/graphs/               ├── src/vector_stores/
└── dejavu/reranker/             └── src/graphs/

cli/python/ ──▶ dejavu-memory (optional, for OSS mode)
cli/node/   ──▶ dejavu-memory (npm, for API calls)
vercel-ai-sdk/ ──▶ ai, @ai-sdk/* providers
openclaw/   ──▶ dejavu-memory (npm)
```

## Development Setup

### Requirements

- **Python**: 3.9+ (3.10+ for CLI)
- **Node.js**: v18+ (v20 or v22 recommended)
- **pnpm**: v10+ (`npm install -g pnpm@10`) — used for all TypeScript packages
- **Hatch**: Python build/environment tool (`pip install hatch`)
- **Docker**: Required for `server/` and `openmemory/` development

### Initial Setup

```bash
# Python SDK
hatch shell dev_py_3_11           # creates environment with all deps
pre-commit install                # install git hooks

# TypeScript packages
cd dejavu-ts && pnpm install        # TS SDK
cd cli/node && pnpm install       # Node CLI
cd vercel-ai-sdk && pnpm install  # Vercel AI provider
cd openclaw && pnpm install       # OpenClaw plugin
```

## Build, Lint, and Test Commands

### Python SDK (`dejavu/`)

```bash
# Environment setup (uses Hatch)
hatch shell dev_py_3_11           # or dev_py_3_9, dev_py_3_10, dev_py_3_12

# Linting and formatting
make lint                          # ruff check
make format                        # ruff format
make sort                          # isort dejavu/

# Tests
make test                          # pytest tests/
make test-py-3.9                   # test specific Python version (3.9–3.12)

# Build and publish
make build                         # hatch build
make publish                       # hatch publish
```

- **Python:** 3.9, 3.10, 3.11, 3.12
- **Linter/formatter:** Ruff (line length **120**)
- **Import sorting:** isort (`profile = "black"`)
- **Test framework:** pytest (with pytest-mock, pytest-asyncio)
- **Pre-commit hooks:** ruff + isort — run `pre-commit install` before committing

### TypeScript SDK (`dejavu-ts/`)

```bash
cd dejavu-ts
pnpm install
pnpm run build                     # tsup
pnpm run test                      # jest (all tests)
pnpm run test:unit                 # jest --coverage (unit tests only)
pnpm run test:integration          # jest (integration tests, needs VENICE_API_KEY)
pnpm run test:ci                   # jest --coverage --ci (CI mode)
pnpm run test:watch                # jest watch mode
```

- **Node:** 20, 22 (CI-tested)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JSingletonAI/dejavu](https://github.com/JSingletonAI/dejavu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
