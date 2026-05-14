---
trigger: always_on
description: Provides UPS monitoring via NUT (Network UPS Tools)
---

# Claude Development Guide for Homelab MCP

## Project Overview

**Repository:** <https://github.com/bjeans/homelab-mcp>
**Docker Hub:** <https://hub.docker.com/r/bjeans/homelab-mcp>
**Version:** 3.0.0 (Released: 2026-01-14)
**License:** MIT
**Purpose:** Open-source MCP servers for homelab infrastructure management through Claude Desktop

**⚠️ Breaking Changes in v3.0:** See [MIGRATION_V3.md](MIGRATION_V3.md) for upgrade guide.

This project provides real-time monitoring and control of homelab infrastructure through 7 specialized MCP servers, including Docker/Podman containers, Ollama AI models, Pi-hole DNS, Unifi networks, UPS monitoring, and Ansible inventory management via the Model Context Protocol.

**Deployment Options:**
- Native Python installation (recommended for development)
- Docker container from Docker Hub: `bjeans/homelab-mcp:latest` (recommended for production)
- Docker build from source (for customization)

## Core Philosophy

1. **Security First** - Never commit credentials, IPs, or sensitive data
2. **Configuration as Code** - All settings via environment variables or Ansible inventory
3. **User Privacy** - All example files use placeholder data
4. **Production-Grade** - Code quality suitable for critical infrastructure
5. **Open Source** - Community-friendly, well-documented, MIT licensed

## Project Structure

```text
homelab-mcp/
├── MCP Servers (7 production servers)
│   ├── ansible_mcp_server.py          # Ansible inventory queries
│   ├── docker_mcp_podman.py           # Docker/Podman container monitoring
│   ├── ollama_mcp.py                  # Ollama AI model management
│   ├── pihole_mcp.py                  # Pi-hole DNS monitoring
│   ├── unifi_mcp_optimized.py         # Unifi network device monitoring
│   ├── ups_mcp_server.py              # UPS/NUT monitoring
│   └── ping_mcp_server.py             # Network connectivity testing
│
├── Unified Server
│   ├── homelab_unified_mcp.py         # Combines all 7 servers
│   ├── mcp_config_loader.py           # Environment variable security
│   └── mcp_error_handler.py           # Centralized error handling
│
├── Configuration & Examples
│   ├── .env.example                   # Configuration template (gitignored)
│   ├── ansible_hosts.example.yml      # Ansible inventory example (gitignored)
│   └── ansible_config_manager.py      # Centralized config loader
│
└── Documentation
    ├── README.md                      # User documentation
    ├── MIGRATION_V3.md                # v2.x → v3.0 upgrade guide
    ├── CONTRIBUTING.md                # Contribution guide & development workflows
    ├── DOCKER.md                      # Docker deployment
    ├── SECURITY.md                    # Security guidelines
    └── CHANGELOG.md                   # Version history
```

## Architecture Patterns

### FastMCP Decorator Pattern (v3.0+)

All servers use **FastMCP's decorator pattern** for simple, pythonic tool definitions. No classes or boilerplate needed!

#### 1. Basic MCP Server Structure

```python
#!/usr/bin/env python3
"""
UPS MCP Server v3.0 (FastMCP)
Provides UPS monitoring via NUT (Network UPS Tools)
"""

import logging
import os
import sys
from pathlib import Path

from fastmcp import FastMCP
from mcp_config_loader import load_env_file

logging.basicConfig(level=logging.INFO, stream=sys.stderr)
logger = logging.getLogger(__name__)

# Initialize FastMCP server
mcp = FastMCP("UPS Monitor")

# Load environment variables
SCRIPT_DIR = Path(__file__).parent
ENV_FILE = SCRIPT_DIR / ".env"

# Only load .env if NOT in unified mode (to avoid duplicate loading)
if not os.getenv("MCP_UNIFIED_MODE"):
    load_env_file(ENV_FILE, allowed_vars={"UPS_*", "NUT_*"}, strict=True)

# Service-specific helper functions
def _load_ups_hosts():
    """Load UPS hosts from Ansible inventory"""
    # Lazy import - only load Ansible when needed (avoids FastMCP import hook conflict)
    from ansible_config_manager import AnsibleConfigManager
    # Implementation here
    ...
```

#### 2. Adding Tools with @mcp.tool() Decorator

```python
from mcp import types

# Simple tool with annotations
@mcp.tool(
    annotations=types.ToolAnnotations(
        readOnlyHint=True,
        destructiveHint=False,
        idempotentHint=False,  # Status changes over time
        openWorldHint=True,
    )
)
def ups_get_status() -> str:
    """Get UPS status from all configured NUT servers"""
    ups_hosts = _load_ups_hosts()

    if not ups_hosts:
        return "No UPS hosts configured"

    output = "=== UPS STATUS ===\n\n"
    for host, config in ups_hosts.items():
        # Query UPS status
        ...
    return output


# Tool with parameters and type hints
@mcp.tool(
    annotations=types.ToolAnnotations(
        readOnlyHint=True,
        destructiveHint=False,
        idempotentHint=False,
        openWorldHint=True,
    )
)
def ups_get_details(host: str, ups_name: str = "") -> str:
    """
    Get detailed UPS information from specific NUT server

    Args:
        host: Hostname of the NUT server to query
        ups_name: Optional specific UPS name (default: first UPS on host)
    """
    ups_hosts = _load_ups_hosts()

    if host not in ups_hosts:
        return f"Unknown host: {host}"

    # Query UPS details
    ...
    return output


# Run server

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bjeans/homelab-mcp](https://github.com/bjeans/homelab-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
