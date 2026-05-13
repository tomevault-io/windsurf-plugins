---
trigger: always_on
description: This project uses the Salesforce Research LLM Gateway proxy for all LLM calls (task generation, evaluation, user simulation, judging).
---

# MCPEval Development Guide

## LLM Inference via SFRGateway

This project uses the Salesforce Research LLM Gateway proxy for all LLM calls (task generation, evaluation, user simulation, judging).

### Starting the gateway

```bash
cd sfrgateway
# First time: copy .env.template to .env and fill in your API key
cp .env.template .env  # then edit .env with your X_API_KEY
PROXY_PORT=8008 uv run python server.py
```

The `.env` file in that directory contains the `X_API_KEY` for the upstream gateway. See `sfrgateway/README.md` for details.

### Model config files

Point any model config JSON at the local proxy:

```json
{
    "model": "gpt-4o-mini",
    "base_url": "http://localhost:8008/v1",
    "api_key": "dummy",
    "temperature": 0.1,
    "max_tokens": 4000
}
```

- `api_key` must be set to any non-empty string (e.g. `"dummy"`) — the real key is handled by the proxy.
- Port 8008 is the default. MCPEval/UserBench historically use port 8009.
- Available models include `gpt-4o-mini`, `gpt-4o`, `gpt-5-nano`, and others listed at `http://localhost:8008/v1/models`.

### Environment variables for CLI

The `OpenAIMCPClient` reads `OPENAI_API_KEY` and `OPENAI_BASE_URL` from the environment (the model config file does NOT pass these to the MCP client). Always set them when running CLI commands:

```bash
export OPENAI_API_KEY=dummy
export OPENAI_BASE_URL=http://localhost:8008/v1
```

Or prefix each command: `OPENAI_API_KEY=dummy OPENAI_BASE_URL=http://localhost:8008/v1 uv run mcp-eval ...`

## Running the project

```bash
uv venv && uv pip install -e ".[dev]"
uv run mcp-eval --help
```

## Key conventions

- Use `uv run` to execute commands (not raw `python`).
- Model configs are JSON files passed via `--model-config`, `--simulator-model-config`, or `--agent-model-config`.
- MCP servers live in `mcp_servers/` and are passed via `--servers mcp_servers/<name>/server.py`.
- npm-based servers are passed by package name: `--servers @modelcontextprotocol/server-memory`.
- All data I/O uses JSONL format (one JSON object per line).
- Generated data (`.jsonl`, `eval_*/`, `demo_simulation/`, `data.db`) is git-ignored — do not commit evaluation outputs.

## Available MCP Servers

### Self-contained (no credentials, deterministic)
| Server | Path | Tools | Description |
|--------|------|-------|-------------|
| hr_management | `mcp_servers/hr_management/server.py` | 10 | Departments, employees, leave, reviews, org chart (embedded SQLite) |
| ecommerce | `mcp_servers/ecommerce/server.py` | 11 | Products, orders, customers, inventory, sales (embedded SQLite) |
| datetime_tools | `mcp_servers/datetime_tools/server.py` | 7 | Timezone, date diff, business days, holidays (pure computation) |
| unit_converter | `mcp_servers/unit_converter/server.py` | 6 | Length, weight, temp, volume, speed, data size (pure computation) |
| special_calculator | `mcp_servers/special_calculator/server.py` | 4 | Demo arithmetic with special transforms |
| sqlite | `mcp_servers/sqlite/server.py` | 8 | General-purpose SQLite operations |
| filesystem | npm: `@modelcontextprotocol/server-filesystem` | 14 | Local file operations |
| memory | npm: `@modelcontextprotocol/server-memory` | 9 | Knowledge graph (entities, relations, search) |

### Public API (free, no auth)
| Server | Path | Tools |
|--------|------|-------|
| book | `mcp_servers/book/server.py` | 8 |
| youtube | `mcp_servers/youtube/server.py` | 4 |
| healthcare | `mcp_servers/healthcare/server.py` | 5 |
| sports | `mcp_servers/sports/server.py` | 4 |

### Credential-required
| Server | Path | Tools | Keys needed |
|--------|------|-------|-------------|
| travel_assistant | `mcp_servers/travel_assistant/server.py` | 6 | `SERPAPI_API_KEY`, `YELP_API_KEY` |
| yfinance | `mcp_servers/yfinance/server.py` | 10 | Yahoo Finance (rate-limited) |
| airbnb | npm: `@openbnb/mcp-server-airbnb` | 2 | None (npm abstracted) |
| national_park | npm: `mcp-server-nationalparks` | 6 | `NPS_API_KEY` (free) |
| crm_bench | `mcp_servers/crm_bench/server.py` | 11 | Stub implementation |

## Complete Evaluation Workflow

### Single-turn pipeline

```bash
# 1. Generate tasks from a server's tools
uv run mcp-eval generate-tasks \
  --servers mcp_servers/hr_management/server.py \
  --model-config model_config.json \
  --num-tasks 10 \
  --output tasks.jsonl

# 2. Verify tasks are executable (agent actually calls the tools)
uv run mcp-eval verify-tasks \
  --servers mcp_servers/hr_management/server.py \
  --tasks-file tasks.jsonl \
  --model-config model_config.json \
  --output verified_tasks.jsonl \
  --non-interactive

# 3. (Optional) Revalidate task descriptions for accuracy
uv run mcp-eval revalidate-tasks \
  --verified-tasks-file verified_tasks.jsonl \
  --model-config model_config.json \
  --output final_tasks.jsonl

# 4. Evaluate a model against the tasks
uv run mcp-eval evaluate \
  --servers mcp_servers/hr_management/server.py \
  --tasks-file verified_tasks.jsonl \
  --model-config model_config.json \
  --output results.json

# 5. Analyze results
uv run mcp-eval analyze \
  --predictions results.json \
  --ground-truth verified_tasks.jsonl
```

### Multi-turn simulation pipeline

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SalesforceAIResearch/MCPEval](https://github.com/SalesforceAIResearch/MCPEval) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
