---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is Gluon Agent?

AI orchestrator for managing multiple Claude Code agents across projects. Provides session persistence, resume capability, workspace-based project discovery, and multiple interfaces (CLI, Telegram, Discord).

## LLM Providers

Gluon supports **four** LLM providers — AWS Bedrock, direct Anthropic API, Google Vertex AI, and Microsoft Foundry (Azure AI Foundry). Selectable via `GLUON_LLM_PROVIDER` env var, the `gluon provider` CLI command, or the web dashboard Settings page. Default is `bedrock`. Provider abstraction lives in `src/gluon/llm_provider.py`; see `docs/LLM-PROVIDER.md` for the full design.

| Tier                | Bedrock Model ID                                     | Anthropic Model ID            | Vertex Model ID                | Foundry Model ID      |
|---------------------|------------------------------------------------------|-------------------------------|--------------------------------|-----------------------|
| claude-opus-4.6     | global.anthropic.claude-opus-4-6-v1                  | claude-opus-4-6               | claude-opus-4-6                | claude-opus-4-6       |
| claude-opus-4.5     | global.anthropic.claude-opus-4-5-20251101-v1:0       | claude-opus-4-5-20251101      | claude-opus-4-5@20251101       | claude-opus-4-5       |
| claude-sonnet-4.6   | global.anthropic.claude-sonnet-4-6                   | claude-sonnet-4-6             | claude-sonnet-4-6              | claude-sonnet-4-6     |
| claude-haiku-4.5    | global.anthropic.claude-haiku-4-5-20251001-v1:0      | claude-haiku-4-5-20251001     | claude-haiku-4-5@20251001      | claude-haiku-4-5      |

**IMPORTANT:** We only support the four tiers listed above. Provider resolution order: explicit argument → `GLUON_LLM_PROVIDER` env var → `llm_provider` setting in the DB → default `bedrock`.

**Subprocess env:** Each provider's `runner_env()` method contributes the `CLAUDE_CODE_USE_*` flag and any required credentials to the Claude Code subprocess. Never hardcode `CLAUDE_CODE_USE_BEDROCK=1` in compose or env files — let the provider emit it.

## Docker Operations

**IMPORTANT: Always use `docker compose` for container operations. NEVER use raw `docker run` commands.**

There are two compose files for different purposes:

| File | Purpose | Image | Container | Env file |
|------|---------|-------|-----------|----------|
| `docker-compose.dev.yml` | Local dev/test — builds from source | `gluon-agent:latest` (local) | `gluon-agent-dev` | `.env.local` |
| `docker-compose.yml` | Production — pulls from GHCR | `ghcr.io/carrotly-ai/gluon-agent:latest` | `gluon-agent` | `.env` |

Both compose files mount critical host directories:
- `~/.claude` - Claude CLI credentials (authentication)
- `~/.gluon` - Gluon database and logs
- `~/.aws` - AWS credentials for Bedrock
- `~/.config/gh` - GitHub CLI for PR operations

### Dev/Test (build from source)

```bash
# Rebuild and restart (after code changes)
docker compose -f docker-compose.dev.yml build && docker compose -f docker-compose.dev.yml up -d

# View logs
docker compose -f docker-compose.dev.yml logs -f

# Stop
docker compose -f docker-compose.dev.yml down

# Full rebuild (no cache)
docker compose -f docker-compose.dev.yml build --no-cache && docker compose -f docker-compose.dev.yml up -d

# Shell into container
docker exec -it gluon-agent-dev bash
```

**After any significant changes** (dependency upgrades, new features, bug fixes), rebuild and redeploy locally:
```bash
docker compose -f docker-compose.dev.yml build && docker compose -f docker-compose.dev.yml up -d
```

### Production (GHCR images)

```bash
# Pull latest and start
docker compose pull && docker compose up -d

# View logs
docker compose logs -f

# Stop
docker compose down

# Shell into container
docker exec -it gluon-agent bash
```

**Port:** 45866 (both dev and production)

## Commands

```bash
# Setup
uv venv && uv pip install -e '.[dev]'

# Run CLI
uv run gluon --help
uv run gluon status
uv run gluon project list
uv run gluon run <project> '<prompt>'

# Run Telegram bot
export GLUON_TELEGRAM_TOKEN="your-token"
uv run gluon bot

# Run Discord bot
export GLUON_DISCORD_TOKEN="your-token"
export GLUON_DISCORD_GUILD="your-guild-id"
uv run gluon discord

# Run both transports
uv run gluon serve --telegram --discord

# Tests
uv run pytest                           # all tests
uv run pytest tests/test_store.py       # single file
uv run pytest tests/test_store.py::test_name -v  # single test

# Linting & Formatting
uv run ruff format .
uv run ruff check .
uv run mypy src/gluon

# Debug database
sqlite3 ~/.gluon/gluon.db
```

## Architecture

```
CLI (cli.py) ───────────────────────────────────────────┐
                                                        │
                Transport Layer (transport/)            │
                ┌─────────────┬─────────────┐           │
                ▼             ▼             ▼           ▼
         TelegramTransport  DiscordTransport  ...    Orchestrator (core.py)
                └─────────────┴─────────────┘           [Business Logic]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [carrotly-ai/gluon-agent](https://github.com/carrotly-ai/gluon-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
