---
trigger: always_on
description: This file provides context for AI coding assistants (Claude Code, Cursor, GitHub Copilot, Codex, etc.) working with the Mem0 repository.
---

# AGENTS.md

This file provides context for AI coding assistants (Claude Code, Cursor, GitHub Copilot, Codex, etc.) working with the Mem0 repository.

## Project Overview

**Mem0** ("mem-zero") is an intelligent memory layer for AI agents and assistants. It provides persistent, personalized memory via both a hosted platform API and self-hosted open-source SDKs.

- **Repository**: https://github.com/mem0ai/mem0
- **Documentation**: https://docs.mem0.ai
- **License**: Apache-2.0

## Repository Structure

This is a **polyglot monorepo** containing Python and TypeScript packages, CLIs, servers, plugins, and documentation.

### Key Directories

| Directory | Description |
|-----------|-------------|
| `mem0/` | Core Python SDK (`mem0ai` on PyPI) — memory, LLMs, embeddings, vector stores, graphs, rerankers |
| `mem0-ts/` | TypeScript SDK (`mem0ai` on npm) — client + OSS memory |
| `cli/python/` | Python CLI (`mem0-cli` on PyPI) — Typer-based, entry point `mem0` |
| `cli/node/` | Node CLI (`@mem0/cli` on npm) — Commander-based, entry point `mem0` |
| `integrations/` | **Agent & editor integrations**, one directory per integration (see "Adding a New Integration") |
| `integrations/mem0-plugin/` | AI editor plugins (Claude Code, Cursor, Codex) — MCP server connection, lifecycle hooks, skills. Contains nested `.opencode-plugin/` (`@mem0/opencode-plugin`) |
| `integrations/openclaw/` | `@mem0/openclaw-mem0` — OpenClaw plugin for Claude Code / AI editors |
| `integrations/pi-agent-plugin/` | `@mem0/pi-agent-plugin` — Pi Agent plugin |
| `integrations/vercel-ai-sdk/` | `@mem0/vercel-ai-provider` — Vercel AI SDK memory provider |
| `server/` | FastAPI REST server for self-hosted Mem0 (Docker: FastAPI + PostgreSQL/pgvector + Neo4j) |
| `openmemory/` | Self-hosted memory platform — `api/` (FastAPI + Alembic + MCP server) and `ui/` (Next.js 15 + React 19) |
| `skills/` | Claude Code skill definitions. Reference skills (SDK knowledge, always-on): `mem0/`, `mem0-cli/`, `mem0-vercel-ai-sdk/`. Pipeline skills (run on demand): `mem0-integrate/`, `mem0-test-integration/`, `mem0-oss-to-platform/` |
| `docs/` | Documentation site (Mintlify) |
| `tests/` | Python SDK tests (pytest) |
| `evaluation/` | Submodule → [`mem0ai/memory-benchmarks`](https://github.com/mem0ai/memory-benchmarks) — benchmarking (LOCOMO, LongMemEval, BEAM) lives in that repo |
| `examples/` | Sample projects & runnable demos — apps, Chrome extension, multi-agent patterns, and Jupyter notebooks (`notebooks/`) |
| `pr-reviews/` | Pull request review materials |
| `scripts/` | Repo-wide utility scripts (e.g., `check-llms-txt-coverage.py` for docs/llms.txt sync) |

### Core Package Dependencies

```
mem0 (Python SDK)          mem0-ts (TypeScript SDK)
├── mem0/memory/           ├── src/client/        (MemoryClient — hosted)
├── mem0/llms/             └── src/oss/           (Memory — self-hosted)
├── mem0/embeddings/           ├── src/llms/
├── mem0/vector_stores/        ├── src/embeddings/
├── mem0/graphs/               ├── src/vector_stores/
└── mem0/reranker/             └── src/graphs/

cli/python/ ──▶ mem0ai (optional, for OSS mode)
cli/node/   ──▶ mem0ai (npm, for API calls)
integrations/vercel-ai-sdk/ ──▶ ai, @ai-sdk/* providers
integrations/openclaw/ ──▶ mem0ai (npm)
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
cd mem0-ts && pnpm install        # TS SDK
cd cli/node && pnpm install       # Node CLI
cd integrations/vercel-ai-sdk && pnpm install  # Vercel AI provider
cd integrations/openclaw && pnpm install       # OpenClaw plugin
```

## Build, Lint, and Test Commands

### Python SDK (`mem0/`)

```bash
# Environment setup (uses Hatch)
hatch shell dev_py_3_11           # or dev_py_3_9, dev_py_3_10, dev_py_3_12

# Linting and formatting
make lint                          # ruff check
make format                        # ruff format
make sort                          # isort mem0/

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

### TypeScript SDK (`mem0-ts/`)

```bash
cd mem0-ts
pnpm install
pnpm run build                     # tsup
pnpm run test                      # jest (all tests)
pnpm run test:unit                 # jest --coverage (unit tests only)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mem0ai/mem0](https://github.com/mem0ai/mem0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
