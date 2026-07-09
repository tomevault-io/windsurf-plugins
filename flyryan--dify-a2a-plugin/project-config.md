---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Dify plugin that implements the Agent-to-Agent (A2A) protocol, allowing Dify agents to communicate with and delegate tasks to external agents. The plugin provides tools for discovering, calling, and managing remote agents through JSON-RPC 2.0.

## Development Commands

### Plugin Development and Testing

```bash
# Install Python dependencies
# Note: Manifest requires Python 3.12, but newer versions may work in development
python3 -m venv .venv
source .venv/bin/activate  # On Mac/Linux
pip install -r requirements.txt

# Run unit tests
python3 -m unittest tools/tests/test_a2a_registry.py

# Build the plugin package (creates .difypkg file)
dify plugin package

# Install the Dify CLI (if not already installed)
brew tap langgenius/dify
brew install dify
```

**Important Notes:**
- Manifest specifies Python 3.12 as runtime requirement
- Local development may work with newer Python versions (e.g., 3.14)
- Production deployment in Dify will use Python 3.12 as specified in manifest

### Local Testing with Dify Plugin Runtime

For testing with the actual Dify plugin runtime environment:

1. **Using Dify CLI debug mode:**
   ```bash
   dify plugin start --debug
   ```

2. **With dify-plugin-daemon** (if available locally):
   - `dify-plugin-daemon` is not included in this repository (gitignored)
   - It's a local reference copy of the Dify plugin runtime
   - See Dify documentation for setting up local daemon

## Architecture Overview

### Plugin Structure

This is a **Tool Provider Plugin** that exposes 5 tools for A2A protocol operations:

1. **list_agents** - Lists all available agents from the registry
2. **get_agent_capabilities** - Fetches agent card from `/.well-known/agent-card.json` (fallback to `agent.json`)
3. **call_agent** - Synchronous agent invocation via `message/send` JSON-RPC method
4. **submit_task** - Asynchronous task submission via `message/stream` with SSE (extracts taskId from first event)
5. **get_task_status** - Check status of async tasks via `tasks/get` JSON-RPC method

### Key Components

```
├── manifest.yaml              # Plugin metadata and configuration
├── main.py                    # Plugin entrypoint (initializes DifyPluginEnv)
├── provider/
│   ├── a2a.yaml              # Provider definition and credentials schema
│   └── a2a.py                # A2AProvider class with credential validation
├── tools/
│   ├── list_agents.py/yaml   # Tool implementations and definitions
│   ├── get_agent_capabilities.py/yaml
│   ├── call_agent.py/yaml
│   ├── submit_task.py/yaml
│   ├── get_task_status.py/yaml
│   └── tests/                # Unit tests with mocked dependencies
├── _assets/                   # Icon/logo assets
│   ├── a2a-logo-black.svg    # Plugin icon (black version with white background)
│   └── a2a-logo-white.svg    # Plugin icon (white version)
├── screenshots/               # Documentation screenshots for README (18 PNG files)
├── LICENSE                    # Apache 2.0 License
├── PRIVACY.md                 # Privacy policy
└── README.md                  # User-facing documentation
```

### Agent Registry Configuration

The plugin uses **individual credential fields** for up to 5 agents, with each agent having:
- `agent_N_name` - Agent identifier (text-input)
- `agent_N_url` - Base URL endpoint (text-input)
- `agent_N_auth_type` - Authentication type (select: none/bearer/api-key/basic)
- `agent_N_api_key` - API key or token (secret-input)
- `agent_N_description` - Agent description (text-input)

Where N = 1 through 5.

This registry is:
- Defined as individual fields in `provider/a2a.yaml`
- Validated in `provider/a2a.py:_validate_credentials()`
- Built at runtime by each tool via `_build_agents_registry()` method
- Accessed from `self.runtime.credentials` in tool implementations

### JSON-RPC 2.0 Protocol

All agent communication follows JSON-RPC 2.0 over HTTP with proper A2A Message object format:

**Synchronous Call (message/send)**:
```json
{
  "jsonrpc": "2.0",
  "method": "message/send",
  "params": {
    "message": {
      "kind": "message",
      "role": "user",
      "messageId": "uuid",
      "parts": [
        {
          "kind": "text",
          "text": "instruction"
        }
      ]
    }
  },
  "id": "uuid"
}
```

**Async Task Submission (message/stream with SSE)**:
```json
{
  "jsonrpc": "2.0",
  "method": "message/stream",
  "params": {
    "message": {
      "kind": "message",
      "role": "user",
      "messageId": "uuid",
      "parts": [
        {
          "kind": "text",
          "text": "instruction"
        }
      ]
    }
  },
  "id": "uuid"
}
```

**SSE Response** (submit_task extracts taskId from first event):
```
event: status-update
data: {"jsonrpc":"2.0","result":{"taskId":"task-123","state":"submitted"},"id":"uuid"}
```

**Task Status (tasks/get)**:
```json
{
  "jsonrpc": "2.0",
  "method": "tasks/get",
  "params": {"id": "task-123"},
  "id": "uuid"
}
```

### Tool Implementation Pattern

All tools follow the same structure:

1. **Inherit from `dify_plugin.Tool`**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flyryan/dify-a2a-plugin](https://github.com/flyryan/dify-a2a-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-09 -->
