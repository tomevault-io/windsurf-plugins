---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

twilio-agent-connect-aws is an open-source library providing AWS-specific integrations for Twilio Agent Connect (TAC). It contains connectors that combine agent runtime integration with multi-channel conversation management.

**Key Architecture**: twilio-agent-connect-aws is a separate package that depends on TAC as an external dependency. It does NOT contain TAC source code - it imports from the `tac` package.

## Understanding TAC

TAC (Twilio Agent Connect) is middleware that integrates with Twilio platform services:
- **Conversation Orchestrator** - Organizes voice/SMS/WhatsApp into conversations
- **Conversation Memory** - Stores customer context, preferences, and history
- **Conversation Intelligence** - Extracts insights via language operators
- **Knowledge** - Semantic search over knowledge bases

**In twilio-agent-connect-aws**: Connectors use TAC to inject memory context into agent prompts and route messages to the appropriate agent instance per conversation.

## Development Commands

```bash
make sync              # Install dependencies (uses uv)
make dev-setup         # Full dev setup with pre-commit hooks
make format            # Format with ruff
make lint              # Lint check only
make type-check        # mypy strict mode
make test              # Run pytest
make check             # All checks (lint + type-check + test)
```

## Package Structure

```
src/tac_aws/
├── __init__.py         # Package exports
├── connectors/         # AWS agent connectors (runtime + channels)
│   ├── strands_connector.py               # StrandsConnector
│   ├── bedrock_connector.py               # BedrockConnector
│   └── bedrock_agentcore_connector.py     # BedrockAgentCoreConnector
├── proxy/              # Lambda proxy handlers (requires agentcore extra)
│   ├── agentcore_lambda.py                # AgentCoreLambdaProxy
│   └── validation.py                      # TwilioSignatureValidator
├── server/             # Server utilities (requires server extra)
│   ├── fastapi_server.py                  # TACAWSFastAPIServer
│   └── agentcore_app.py                   # TACAgentCoreApp (requires agentcore extra)
└── tools/              # LLM tools for Strands
    └── strands.py                         # Memory tool for Strands agents

getting_started/examples/   # Full working examples (FastAPI servers)
deploy/                     # Production deployment guides
```

## Code Conventions

- **Python 3.10+**: Use `typing` module types (`List`, `Dict`, `Optional`)
- **mypy strict**: All functions need type hints, no incomplete defs
- **ruff**: Line length 100, black-compatible formatting
- **Imports from TAC**: Always import from `tac` package, never from internal `tac_aws` except for local imports

## Dependencies

### Core Dependency

```toml
dependencies = [
    "twilio-agent-connect>=1.0.0,<2",
]
```

### Optional Dependencies

- `strands` - AWS Strands SDK
- `bedrock` - AWS Bedrock Agents (boto3 + type stubs)
- `agentcore` - AWS Bedrock AgentCore (bedrock-agentcore + boto3 + type stubs)
- `server` - FastAPI server utilities (requires `tac[server]`)
- `dev` - Development tools (pytest, ruff, mypy, type stubs)

## Key Concepts

### Connectors

Connectors combine agent runtime integration with multi-channel conversation management:
- Create and manage per-conversation agent instances
- Create Voice and SMS channels
- Inject TAC memory context using `MemoryPromptBuilder`
- Route responses to appropriate channels
- Register with TAC via `on_message_ready()` callback

**StrandsConnector**: AWS Strands SDK integration with per-conversation agent management and SessionManager support.

**BedrockConnector**: AWS Bedrock Agents integration for console-created agents with managed service.

**BedrockAgentCoreConnector**: AWS Bedrock AgentCore integration for custom agent code (Strands, LangGraph, OpenAI SDK). Supports both HTTP and WebSocket runtimes.

### Server Utilities

**TACAWSFastAPIServer**: FastAPI server with AWS ALB header fixing for Twilio signature validation.

**TACAgentCoreApp**: TAC adapter for AgentCore runtime. Registers HTTP (SMS) and WebSocket (Voice) handlers.

**TACAgentCoreWebSocketAdapter**: WebSocket wrapper that sends welcome greeting for Twilio ConversationRelay.

### Proxy Utilities (Lambda)

**AgentCoreLambdaProxy**: Routes Twilio webhooks to AgentCore runtime. Handles signature validation, voice TwiML generation, and webhook forwarding.

**TwilioSignatureValidator**: Webhook signature validation for Lambda events (form-encoded and JSON).

## Import Patterns

### Correct Imports

```python
# TAC imports - external dependency
from tac.core import TAC, TACConfig
from tac.models.session import ConversationSession
from tac.server import TACFastAPIServer

# twilio-agent-connect-aws imports - local package
from tac_aws.connectors import StrandsConnector, BedrockAgentCoreConnector
from tac_aws.proxy import AgentCoreLambdaProxy, TwilioSignatureValidator
from tac_aws.server import TACAgentCoreApp, TACAWSFastAPIServer
```

### Incorrect Imports (DO NOT DO)

```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [twilio/twilio-agent-connect-aws](https://github.com/twilio/twilio-agent-connect-aws) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
