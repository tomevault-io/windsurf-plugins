---
trigger: always_on
description: A multi-channel gateway that bridges messaging platforms (Slack, WhatsApp, Google Chat, Discord, Telegram, Email) to any A2A-compliant agent via the Agent-to-Agent (A2A) protocol (JSON-RPC 2.0 over HTTP + SSE).
---

# GEMINI.md — A2A Gateway Agent Guidelines

## Project Overview

A multi-channel gateway that bridges messaging platforms (Slack, WhatsApp, Google Chat, Discord, Telegram, Email) to any A2A-compliant agent via the Agent-to-Agent (A2A) protocol (JSON-RPC 2.0 over HTTP + SSE).

- **Language:** Python 3.12+
- **Framework:** FastAPI with uvicorn
- **Async:** Full asyncio — no blocking calls on the event loop

## Commands

```bash
uv sync                                    # Install dependencies
uv run python -m gateway                   # Start the gateway
uv run pytest                              # Run tests (skips live tests)
uv run pytest -m live                      # Run live integration tests
uv run ruff check gateway/ tests/          # Lint
uv run ruff format gateway/ tests/         # Format
```

## Project Structure

```
gateway/
  channels/          # Channel adapters (one file per platform)
    slack.py, whatsapp.py, google_chat.py, discord.py, telegram.py, email.py
  core/              # Shared infrastructure
    channel.py       # ChannelAdapter base class
    types.py         # InboundMessage, OutboundMessage, Attachment dataclasses
    router.py        # Message pipeline (dispatch → A2A → response)
    a2a_client.py    # HTTP + SSE client for A2A protocol
    rate_limit.py    # RateLimiter + RetryWithBackoff
    concurrency.py   # Per-conversation/user/global semaphore limiter
    debounce.py      # Message coalescing within time window
    chunking.py      # Split long responses for channel limits
    session.py       # Session state with TTL-based sweep
    health.py        # Health monitoring and readiness
    typing_indicator.py
    capabilities.py  # A2A agent card discovery
    policies.py      # Group chat policy enforcement
    markdown.py      # Markdown adaptation per channel
    media.py         # Attachment handling
    logging.py       # Structured logging setup
    interactive.py, interactive_callbacks.py  # Buttons, cards, selects
    ack.py           # Acknowledgment (reactions, read receipts)
  config.py          # Dataclass-based configuration (YAML + env vars)
  server.py          # FastAPI app factory, lifespan, endpoint wiring
tests/
  unit/              # Isolated component tests
  integration/       # Router pipeline + server endpoint tests
  contracts/         # Adapter interface contract tests
  live/              # Real service tests (require credentials)
  helpers/           # MockAdapter, fake A2A server
  conftest.py        # Shared fixtures
```

## Architecture

### Adapter Pattern

Every channel implements `ChannelAdapter` (`gateway/core/channel.py`):

```python
class MyAdapter(ChannelAdapter):
    channel_type = "my_channel"           # Required: sets the adapter name

    def __init__(self, ..., account_id: str = "default") -> None:
        super().__init__(account_id=account_id)  # Must call with account_id

    async def start(self) -> None: ...    # Connect / start polling
    async def stop(self) -> None: ...     # Graceful shutdown
    async def send(self, message: OutboundMessage) -> str | None: ...
```

Optional overrides: `edit_message()`, `send_typing()`, `send_ack()`, `supports_editing` property.

The base class provides `name` (returns `"my_channel"` or `"my_channel:account_id"`), `dispatch()` (forwards inbound messages to the router), and `on_message` callback slot.

### Router Pipeline

`Router` (`gateway/core/router.py`) wires everything together:

1. Adapter registers via `router.register(adapter)`
2. Router sets `adapter.on_message` to its pipeline (optionally wrapped with ack, debounce)
3. On inbound message: group policy check → session lookup → typing indicator → A2A call (streaming or non-streaming) → chunk response → rate-limited send

### Configuration

Dataclass-based config in `gateway/config.py`:

- `GatewayConfig` is the top-level container
- Feature configs: `ChunkingConfig`, `DebounceConfig`, `RateLimitingConfig`, `ConcurrencyConfig`, etc.
- Account configs: `SlackAccountConfig`, `WhatsAppAccountConfig`, etc.
- Loading: YAML file parsed first, then env var overrides applied via `_apply_env_overrides()`
- Unknown config keys produce a warning (not a crash)
- Helper `_build(cls, data)` filters dict keys to match dataclass fields

Env var convention: `SLACK_BOT_TOKEN`, `WHATSAPP_ACCESS_TOKEN`, `A2A_SERVER_URL`, `GATEWAY_PORT`.

## Code Style

### Essentials

- Every module starts with `from __future__ import annotations`
- Type hints on all function parameters and return types
- Union syntax: `str | None` (not `Optional[str]`)
- Line length: **90 characters** (configured in ruff)
- Imports: absolute only (`from gateway.core.types import InboundMessage`)
- Use `TYPE_CHECKING` guard for imports only needed by type checkers

### Naming

| Thing | Convention | Example |
|-------|-----------|---------|
| Classes | PascalCase | `ChannelAdapter`, `RateLimiter` |
| Functions / methods | snake_case | `send_message`, `_handle_inner` |
| Constants | UPPER_SNAKE | `RETRYABLE_EXCEPTIONS`, `GRAPH_API` |
| Private members | leading underscore | `_http`, `_lock`, `_sessions` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eliasecchig/a2a-gateway](https://github.com/eliasecchig/a2a-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
