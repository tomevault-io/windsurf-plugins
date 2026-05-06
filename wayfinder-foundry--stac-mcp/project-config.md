---
trigger: always_on
description: STAC MCP Server is a Python-based Model Context Protocol (MCP) server that provides access to STAC (SpatioTemporal Asset Catalog) APIs for geospatial data discovery and access. It enables AI assistants to search for satellite imagery, weather data, and other geospatial datasets.
---

# STAC MCP Server Development Guide

STAC MCP Server is a Python-based Model Context Protocol (MCP) server that provides access to STAC (SpatioTemporal Asset Catalog) APIs for geospatial data discovery and access. It enables AI assistants to search for satellite imagery, weather data, and other geospatial datasets.

**Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

## Working Effectively

### Bootstrap, Build, and Test Repository
Run these commands in order to set up a working development environment:

```bash
# Install dependencies (takes ~15 seconds, NEVER CANCEL, set timeout to 120+ seconds)
# May fail due to network timeouts in sandbox environments
pip install -e ".[dev]"

# Run tests (takes ~1 second, NEVER CANCEL, set timeout to 30+ seconds)
pytest

# Run formatting and linting (takes ~0.2 seconds total)
ruff format stac_mcp/ tests/
ruff check stac_mcp/ tests/ --fix
```

- After ANY code edit (even small), re-run `ruff format` and `ruff check --fix` locally before committing to keep diffs clean and surface issues early.

### Run the MCP Server
```bash
# Run as MCP server (stdin/stdout transport, waits for MCP protocol messages)
stac-mcp

# Test server functionality with example script (takes ~1 second)
python examples/example_usage.py
```

### Key Timing Expectations
- **NEVER CANCEL**: Set timeout to 120+ seconds for pip install (may timeout due to network issues)
- **NEVER CANCEL**: Set timeout to 30+ seconds for pytest (runs in ~1 second)
- Ruff formatting: ~0.2 seconds
- Ruff linting: ~0.02 seconds
- Example script: ~0.6 seconds
- MCP server timeout test: 5 seconds (expected timeout with exit code 124)

## Validation

### Always Run These Validation Steps After Making Changes
**MUST** run these commands and ensure they pass before every `git commit` and `git push`.

1.  **Format Code:**
    ```bash
    uv run ruff format stac_mcp/ tests/
    ```
2.  **Lint and Auto-Fix:**
    ```bash
    uv run ruff check stac_mcp/ tests/ --fix --no-cache
    ```
3.  **Run Tests:**
    ```bash
    uv run pytest -q
    ```
4.  **Verify Test Coverage:**
    ```bash
    uv run coverage run -m pytest -v
    uv run coverage report --fail-under=85
    ```

**IMPORTANT:** Run the `ruff format` and `ruff check` commands multiple times in sequence to ensure all issues are resolved, as auto-fixing can sometimes introduce new formatting needs.

3. **Manual MCP server test** (optional, server will wait for input):
   ```bash
   timeout 5s stac-mcp || true  # Should timeout after 5s (exit code 124), indicating server is running
   ```

### End-to-End Validation Scenarios
After making changes, always test these core workflows:

1. **Tool Discovery**: Verify all 4 tools are available
   - search_collections
   - get_collection  
   - search_items
   - get_item
   - estimate-data-size

2. **Network Scenarios**: The server handles network errors gracefully (no real network access needed for testing)

## Use public APIs — do not reach into package internals

- Always prefer public, documented APIs from third-party libraries. Do not read, write, or rely on underscore-prefixed or private attributes (for example avoid accessing `client._stac_io`, `session._pool`, or other internals that are not part of the published API).
- For STAC interactions, prefer the public `pystac_client.Client` and `StacApiIO` (or a custom `StacIO` subclass) and call public methods such as `client.search()`, `client.get_collection()`, and `client.get_item()`.
- If you need functionality not exposed by the public client, either:
  - implement and pass a custom `StacIO` subclass to the public client constructor, or
  - perform an explicit HTTP request using `requests` with clearly-set headers and timeouts (preferred to monkey-patching internals).
- For testability, prefer dependency injection (accept `stac_io`, `requests.Session`, or a `client` parameter) so tests can inject fakes/mocks without touching internals.
- Examples (preferred patterns):

  ```py
  # Preferred: public Client with a StacIO instance
  from pystac_client import Client
  from pystac_client.stac_api_io import StacApiIO

  stac_io = StacApiIO(headers={"X-API-Key": "..."})
  client = Client.open("https://example.com/stac", stac_io=stac_io)
  results = client.search(collections=["c1"]).items()
  ```

  ```py
  # Or, when you need a direct HTTP call for a capability not exposed publicly
  import requests

  resp = requests.post(
      "https://example.com/stac/search",
      json={"collections": ["c1"]},
      headers={"Accept": "application/json"},
      timeout=30,
  )
  ```

Following these patterns improves maintainability and reduces brittle dependencies on upstream implementation details.

## Project Structure

### Repository Root
```
.
├── .github/                 # GitHub configuration
├── .gitignore              # Python gitignore
├── LICENSE                 # Apache 2.0 license  
├── README.md               # User documentation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Wayfinder-Foundry/stac-mcp](https://github.com/Wayfinder-Foundry/stac-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
