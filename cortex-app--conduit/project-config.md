---
trigger: always_on
description: Conduit is a Model Context Protocol (MCP) server that provides seamless integration with Phabricator and Phorge APIs through a modular client pattern with unified entry points.
---

# Conduit MCP Server - AI Development Guide

## Architecture Overview

Conduit is a Model Context Protocol (MCP) server that provides seamless integration with Phabricator and Phorge APIs through a modular client pattern with unified entry points.

### Core Architecture
- **Main Server** (`src/conduit.py`): FastMCP-based server with dual transport modes (stdio/HTTP-SSE)
- **Unified Client** (`src/client/unified.py`): Enhanced client with caching, retries, and token optimization
- **Modular Clients** (`src/client/*.py`): Specialized clients for different Phabricator APIs

### Supported APIs
- **Maniphest**: Task management (search, create, edit tasks, get transaction history)
- **Differential**: Code review (search, create, manage revisions)
- **Diffusion**: Repository management (search, browse, commits, file content with base64 decoding)
- **File**: File management (search, upload, download)
- **User**: User management (information, queries)
- **Project**: Project management (search, members, workboards)
- **Conduit**: System interface (ping, capabilities, info)

## Development Workflow

### Environment Setup

This project uses `uv` virtual environment. Before executing any Python code, activate the Python virtual environment first:
```bash
# Since this is a Phabricator MCP Server, you may set environment variables
# Example environment variables (replace with your actual values):
# export PHABRICATOR_TOKEN="your-32-character-token"
# export PHABRICATOR_URL="https://your-phabricator-instance.com/api/"
# export PHABRICATOR_PROXY="socks5://127.0.0.1:1080"  # Optional
# export PHABRICATOR_DISABLE_CERT_VERIFY=1  # Optional (security risk)

# Actual execution with environment variables:
PHABRICATOR_TOKEN="your-32-character-token" \
PHABRICATOR_URL="https://your-phabricator-instance.com/api/" \
PHABRICATOR_PROXY="socks5://127.0.0.1:1080" \
PHABRICATOR_DISABLE_CERT_VERIFY=1 \
source venv/bin/activate && python your_script.py

# Example, actual name may changed
source venv/bin/activate && uv pip xxx
source xxx && python xxx
```

### MCP Server Execution Modes

The Conduit MCP Server supports two execution modes:

#### 1. Stdio Mode (Default)
- **Required**: `PHABRICATOR_TOKEN`, `PHABRICATOR_URL`
- **Run**: `python run.py`

#### 2. HTTP/SSE Mode
- **Required**: `PHABRICATOR_URL` only
- **Token**: Provided via HTTP header `X-PHABRICATOR-TOKEN`
- **Run**: `python run.py --host 127.0.0.1 --port 8000`

### Testing
To run unittests, you need to install Docker on your environment to run Phorge in the background.
```bash
cd tests
docker build -t phorge_debug .
docker run -d --rm -p 8080:80 --name phorge_debug phorge_debug
```
Use `docker ps` to determine whether the user has run the phorge image already. The phorge_debug container will automatically initialize the Phorge environment, enable username/password authorization, and generate a User API key for the default admin user with password `Passw0rd`. By default, you can access your locally running Phorge instance at http://127.0.0.1:8080/.

You can retrieve the User API Key by entering the following command:
```bash
docker exec phorge_debug /usr/local/bin/get-api-token.sh
```

Before executing any Python code or command, you need to activate the Python virtual environment first. Use `source xxx/bin/activate` to activate venv. After that, you can run unittest (or any Python command) locally by this command:
```bash
PHABRICATOR_TOKEN=<api-token> PHABRICATOR_URL=http://127.0.0.1:8080/api/ pytest # or any Python command
```

**Note on Testing Strategy**: The unit tests in this project are designed to test against live Phabricator/Phorge instances without mocking. This ensures all code works correctly with real API responses. Tests accept a `PHABRICATOR_URL` and `PHABRICATOR_TOKEN` environment variable to connect to the test instance. All tests in `conduit/client/tests/` and `conduit/tests/` are integration tests that verify real API behavior.

### Code Quality Tools
- **Security**: bandit scanning (`.bandit_scan.cfg`)
- **Pre-commit**: Automated quality checks (`.pre-commit-config.yaml`)

## API Patterns

### Client Usage Patterns
```python
# Basic client (backward compatible)
client = PhabricatorClient(api_url, api_token)

# Enhanced client with caching/retries
client = PhabricatorClient(
    api_url, api_token,
    timeout=60.0,
    max_retries=5,
    enable_cache=True,
    cache_ttl=600
)

# Access specialized modules
tasks = client.maniphest.search_tasks(constraints={"statuses": ["open"]})
diffs = client.differential.search_revisions(author_phids=[user_phid])
```

### MCP Tool Development
- All tools use `@handle_api_errors` decorator for structured error responses
- Apply `@optimize_token_usage` for search results that may return large datasets
- Use type-safe transaction objects from `conduit.client.types` for updates
- Follow naming convention: `pha_<module>_<action>` (e.g., `pha_task_search_advanced`)

### Differential Tools Usage Patterns
```python
# Get revision with complete diff history
result = pha_diff_get("D123")
revision = result["revision"]
current_diff_phid = revision["fields"]["diffPHID"]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cortex-app/conduit](https://github.com/cortex-app/conduit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
