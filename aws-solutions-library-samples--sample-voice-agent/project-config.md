---
trigger: always_on
description: - Use Mermaid diagrams instead of ASCII art for all architecture diagrams, flow charts, and sequence diagrams in Markdown files.
---

## Documentation Conventions

- Use Mermaid diagrams instead of ASCII art for all architecture diagrams, flow charts, and sequence diagrams in Markdown files.

## Logging Conventions

All Python code in this project uses **structlog** for structured logging. Do not use stdlib `logging` or `loguru`.

**Setup pattern:**
```python
import structlog

logger = structlog.get_logger(__name__)
```

**Log call style** — use snake_case event names as the first argument, then keyword arguments for structured data. Never use printf-style `%s` formatting or f-strings in log messages:
```python
# Correct
logger.info("tool_execution_complete", tool_name=name, execution_time_ms=42.1, call_id=cid)
logger.warning("task_protection_enable_failed", note="Accepting call without protection")
logger.error("pipeline_error", error=str(e), error_type=type(e).__name__)

# Wrong — do not use these patterns
logger.info("Tool %s completed in %.1fms", name, elapsed)   # printf style
logger.info(f"Tool {name} completed in {elapsed:.1f}ms")     # f-string style
```

**For capability agents** (separate containers): add `structlog` to `requirements.txt` and configure it at the top of the entrypoint:
```python
import structlog

structlog.configure(
    processors=[
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.JSONRenderer(),
    ]
)
logger = structlog.get_logger(__name__)
```

The voice agent's `service_main.py` already configures structlog globally with `contextvars` support for per-call context binding.

## Container Builds

Use `finch` instead of Docker for all container image builds and deployments. For CDK deployments, set the `CDK_DOCKER` environment variable to `finch` so that CDK asset bundling uses finch:

```bash
export CDK_DOCKER=finch
```

## AWS Configuration

Configure your AWS CLI with credentials for the target account before deploying:

```bash
aws configure --profile <your-profile>
export AWS_PROFILE=<your-profile>
```

## Environment Variables

### Observability Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `ENVIRONMENT` | `production` | Deployment environment (poc, dev, staging, prod) - used as CloudWatch dimension |
| `ENABLE_AUDIO_QUALITY_MONITORING` | `true` | Enable AudioQualityObserver for RMS/peak/silence metrics |
| `ENABLE_CONVERSATION_LOGGING` | `false` | Enable ConversationObserver for transcript logging |
| `ENABLE_TOOL_CALLING` | `true` | Enable LLM tool calling (function calling) for executing actions |
| `ENABLE_FILLER_PHRASES` | `true` | Enable filler phrases during tool execution delays |
| `FILLER_DELAY_THRESHOLD_MS` | `1500` | Milliseconds to wait before playing filler phrase |

### LLM Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `LLM_MODEL_ID` | `us.anthropic.claude-haiku-4-5-20251001-v1:0` | Bedrock model/inference profile ID for LLM processing. Also configurable via SSM at `/voice-agent/config/llm-model-id` |

### Capability Agent Registry (A2A) Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `A2A_NAMESPACE` | *required* | CloudMap HTTP namespace name for agent discovery - set by CDK |
| `A2A_CACHE_TTL_SECONDS` | `60` | Response cache TTL for A2A tool results |
| `A2A_CACHE_MAX_SIZE` | `100` | Max cached A2A responses per tool handler |

Feature flag via SSM at `/voice-agent/config/enable-capability-registry` (default: `false`). When enabled, the voice agent discovers capability agents (KB, CRM) via CloudMap and registers their skills as LLM tools via the A2A protocol.

For a step-by-step guide on adding a new capability agent, see [Adding a Capability Agent](docs/guides/adding-a-capability-agent.md). For architecture details, see [Capability Agent Pattern](docs/patterns/capability-agent-pattern.md).

Additional SSM config:
- `/voice-agent/a2a/poll-interval-seconds` (default: `30`) - CloudMap polling interval
- `/voice-agent/a2a/tool-timeout-seconds` (default: `30`) - A2A call timeout

### Knowledge Base Agent Configuration

These env vars are set on the **KB capability agent** container (not the voice agent):

| Variable | Default | Description |
|----------|---------|-------------|
| `KB_KNOWLEDGE_BASE_ID` | *required* | Bedrock Knowledge Base ID - set by CDK deployment |
| `KB_RETRIEVAL_MAX_RESULTS` | `3` | Maximum document chunks to retrieve per query (1-10) |
| `KB_MIN_CONFIDENCE_SCORE` | `0.3` | Minimum confidence threshold for results (0.0-1.0) |

The KB agent uses a `DirectToolExecutor` that bypasses the inner Strands LLM, calling `search_knowledge_base` directly. This reduces A2A tool latency from ~2,742ms to ~323ms.

### Auto-Scaling Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `MAX_CONCURRENT_CALLS` | `10` | Maximum concurrent calls per container before `/ready` returns 503 and NLB stops routing new calls. Driven by `sessionCapacityPerTask` in CDK config. |

Scaling behavior is configured via CDK context or environment variables at deploy time:

| Parameter | Default | Env Var | Description |
|-----------|---------|---------|-------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws-solutions-library-samples/sample-voice-agent](https://github.com/aws-solutions-library-samples/sample-voice-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
