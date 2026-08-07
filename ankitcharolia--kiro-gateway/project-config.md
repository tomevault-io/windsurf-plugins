---
trigger: always_on
description: This document orients AI agents (Claude, GPT, etc.) contributing to the Kiro
---

# AGENTS.md — Guide for AI Agents Working in Kiro Gateway

This document orients AI agents (Claude, GPT, etc.) contributing to the Kiro
Gateway codebase. A shorter quick-start lives in [`CLAUDE.md`](CLAUDE.md);
user-facing docs live in [`README.md`](README.md) and [`docs/`](docs/).

> If anything here disagrees with the code, the code wins. Read the source and
> update this file.

## Project Philosophy

**Kiro Gateway is a compliance-first bridge to the official `kiro-cli` binary.**

Every completion is fulfilled by talking to `kiro-cli acp` over the Agent
Client Protocol (ACP, JSON-RPC 2.0 on stdio). The gateway **never** calls
private Kiro HTTP endpoints, **never** pools accounts, and **never** touches
credentials — all authentication lives inside `kiro-cli` (`kiro-cli login`).
See [`COMPLIANCE.md`](COMPLIANCE.md).

The gateway's value is **protocol translation**: it lets tools that only speak
the OpenAI or Anthropic API (or native ACP) drive a single Kiro subscription.

### Core Principles

1. **Compliance first.** Only the official binary is invoked. No reverse-
   engineered APIs, no credential handling, no account pooling. A startup guard
   (`kiro.compliance.validate_single_account_compliance`) enforces a single
   active session.
2. **Faithful translation.** Preserve the caller's intent. Translate request
   and response shapes between OpenAI/Anthropic/ACP without silently dropping
   or rewriting user content.
3. **Stateless per request.** Each HTTP request opens a fresh ACP session
   (`session/new`) so concurrent requests stay isolated.
4. **Systems over patches.** Build abstractions that handle a whole class of
   cases rather than one-off hacks.
5. **Paranoid testing.** Every change ships with tests that try to break the
   code — edge cases, error paths, malformed input — not just the happy path.
   The full suite is network-isolated and never spawns the real binary.
6. **Code quality.** English-only identifiers/comments/docstrings; mandatory
   type hints; Google-style docstrings (Args/Returns/Raises); loguru logging at
   decision points; no bare `except:` — catch specific exceptions with context;
   no placeholders — every function is production-ready when committed.
7. **Complete feature consistency.** New behaviour must land in **both** the
   OpenAI and Anthropic shims and in **both** streaming and non-streaming paths,
   with ACP-route coverage where relevant.

### Code Review Reality Check

We can tell low-effort work: missing tests, non-English identifiers, changes to
only one API or one mode, or code that ignores existing patterns. Such PRs face
heavy scrutiny. What gets merged: comprehensive tests, consistency across both
APIs and both modes, and evidence you read the surrounding code.

## Project Overview

- **Language**: Python 3.14+
- **Framework**: FastAPI + uvicorn
- **License**: AGPL-3.0
- **Entry point**: `main.py`
- **Package**: `kiro/`

### Request path

```
OpenAI / Anthropic / native-ACP client
        │  HTTP
        ▼
routes_openai_shim.py / routes_anthropic_shim.py / routes_acp.py
        ▼
shim_service.py        # session lifecycle + event passthrough / aggregation
        ▼
acp_client.py          # one kiro-cli subprocess, JSON-RPC 2.0 over stdio
        ▼
kiro-cli acp           # official, authenticated binary
        ▼
Kiro Backend
```

## Essential Commands

```bash
# Run (bare metal) — needs uv: https://docs.astral.sh/uv/
uv sync                       # creates .venv and installs deps
cp .env.example .env          # set KIRO_GATEWAY_API_KEY
kiro-cli login                # once
uv run main.py                # http://localhost:8000  (--host / --port to override)

# Tests
uv run pytest -q              # full suite (network-isolated, fast)
uv run pytest tests/unit/ -v
uv run pytest tests/integration/ -v
uv run pytest tests/unit/test_acp_compliance.py tests/unit/test_compliance.py -v
uv run pytest --cov=kiro --cov-report=term-missing

```

## Project Structure

```
kiro-gateway/
├── main.py                     # App + lifespan: load .env, start ACPClient, initialize once
├── kiro/
│   ├── acp_client.py           # kiro-cli subprocess + JSON-RPC bridge + protocol translation
│   ├── acp_models.py           # Pydantic models (JSON-RPC envelopes, prompt params, content blocks)
│   ├── shim_service.py         # Per-request session/new, streaming passthrough, aggregation
│   ├── routes_openai_shim.py   # /v1/chat/completions, /v1/models
│   ├── routes_anthropic_shim.py# /v1/messages, /v1/models
│   ├── routes_acp.py           # /acp/chat, /acp/chat/stream
│   ├── config.py               # Env-driven settings (settings object + module constants)
│   ├── compliance.py           # Single-account enforcement at startup
│   └── capability_executor.py  # Stub capability dispatch (retained; not in the live path)
├── tests/                      # conftest.py + unit/ + integration/
├── docs/                       # Translated user docs (en, es, pt, id, zh, ja, ko, ru)
├── .env.example                # Configuration template
├── pyproject.toml              # Project metadata, deps, tool config (uv-managed)
└── uv.lock                     # Pinned dependency lockfile
```

> Some legacy modules from an earlier design may still exist in `kiro/`. The

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ankitcharolia/kiro-gateway](https://github.com/ankitcharolia/kiro-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
