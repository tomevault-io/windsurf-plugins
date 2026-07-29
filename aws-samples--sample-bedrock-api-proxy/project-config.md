---
trigger: always_on
description: **Anthropic-Bedrock API Proxy** — a FastAPI service that translates between the Anthropic Messages API format and AWS Bedrock's APIs. Clients using the Anthropic Python SDK can seamlessly access any Bedrock model.
---

## Project Overview

**Anthropic-Bedrock API Proxy** — a FastAPI service that translates between the Anthropic Messages API format and AWS Bedrock's APIs. Clients using the Anthropic Python SDK can seamlessly access any Bedrock model.

**Key Insight**: Bidirectional translation middleware. Requests: Anthropic format → Bedrock format → Bedrock API → Bedrock response → Anthropic format.

## Development Setup

```bash
# Install
uv sync                    # or: pip install -e ".[dev]"
cp .env.example .env       # configure AWS credentials + settings

# Setup
uv run scripts/setup_tables.py
uv run scripts/create_api_key.py --user-id dev-user --name "Development Key"

# Run
uv run uvicorn app.main:app --reload                           # dev
uv run uvicorn app.main:app --host 0.0.0.0 --port 8000 -w 4   # prod
docker-compose up -d                                            # full stack

# Test
uv run pytest                                    # all tests
uv run pytest --cov=app --cov-report=html        # with coverage
uv run pytest -m integration                     # integration only

# Code quality
black app tests && ruff check app tests && mypy app
```

## Architecture

### Dual API Mode

- **InvokeModel API** (Claude models): Native Anthropic format, minimal conversion, full beta feature support
- **Converse API** (non-Claude models): Requires format conversion, unified API for all Bedrock models
- **OpenAI Chat Completions API** (non-Claude models, optional): When `ENABLE_OPENAI_COMPAT=True`, non-Claude models use Bedrock's OpenAI-compatible endpoint via bedrock-mantle instead of Converse API
- **OpenAI Passthrough** (any model bedrock-mantle accepts, optional): When `ENABLE_OPENAI_PASSTHROUGH=True`, mounts `/openai/v1/{chat/completions,responses,responses/{id},models}` for clients using OpenAI-format directly.

**API selection**: If model ID contains "anthropic" or "claude" → InvokeModel; else if `ENABLE_OPENAI_COMPAT` → OpenAI Chat Completions; else → Converse. OpenAI Passthrough routes are independent and mount at `/openai/v1/*`.

**Multi-Provider Gateway** (optional, `MULTI_PROVIDER_ENABLED`): when enabled, a routing engine (`app/routing/`) selects a target model/provider per request (rule/cost/quality/smart routing), a key pool (`app/keypool/`) rotates encrypted provider keys with rate-limit cooldown + cross-model failover, and `app/compression/` optionally compresses agent context. All flags default off (except `FAILOVER_ENABLED`/`CACHE_AWARE_ROUTING_ENABLED`) — zero impact when `MULTI_PROVIDER_ENABLED=False`. See [docs/smart-routing-guide.md](docs/smart-routing-guide.md).

> **Detailed conversion flows, content block mapping, and streaming implementation**: see [docs/architecture/detailed-flows.md](docs/architecture/detailed-flows.md)

### Configuration

All config in `app/core/config.py` (Pydantic Settings, loads from env vars / `.env`). When adding new features, add corresponding feature flags and config options.

### DynamoDB Tables

| Table | Purpose |
|-------|---------|
| `anthropic-proxy-api-keys` | API keys, budgets, rate limits |
| `anthropic-proxy-usage` | Per-request usage logs |
| `anthropic-proxy-usage-stats` | Aggregated token counts |
| `anthropic-proxy-model-pricing` | Model pricing data |
| `anthropic-proxy-model-mapping` | Anthropic → Bedrock model ID mapping |
| `anthropic-proxy-beta-headers` | Anthropic → Bedrock beta header mappings |
| `anthropic-proxy-response-context` | OpenAI Responses API passthrough context store |
| `anthropic-proxy-providers` | Multi-provider: Bedrock account/provider definitions |
| `anthropic-proxy-provider-keys` | Multi-provider: encrypted provider API keys (key pool) |
| `anthropic-proxy-routing-rules` | Multi-provider: routing rules |
| `anthropic-proxy-failover-chains` | Multi-provider: cross-model failover chains |
| `anthropic-proxy-smart-routing-config` | Multi-provider: RouteLLM smart-routing config |

> **Full schema, budget computation, and aggregation details**: see [docs/architecture/detailed-flows.md](docs/architecture/detailed-flows.md)

## Project Structure

```
app/
├── api/              # Route handlers (thin); includes openai_passthrough/ subpackage
├── converters/       # Anthropic↔Bedrock + Anthropic↔OpenAI conversion logic
├── core/             # Configuration, logging, metrics, security_validator
├── db/               # DynamoDB client and managers (incl. provider_manager, beta_header_cache)
├── middleware/       # Auth and rate limiting
├── schemas/          # Pydantic models (anthropic.py, bedrock.py, provider.py, web_search.py, web_fetch.py, ptc.py)
├── services/         # Business logic, Bedrock calls, provider abstraction; ptc/ web_search/ web_fetch/ subpackages
├── routing/          # Multi-provider routing engine (rule/cost/quality/smart)
├── keypool/          # Multi-provider API-key pool: rotation, failover, encryption
├── compression/      # Agent context compression
└── tracing/          # OpenTelemetry distributed tracing
admin_portal/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws-samples/sample-bedrock-api-proxy](https://github.com/aws-samples/sample-bedrock-api-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
