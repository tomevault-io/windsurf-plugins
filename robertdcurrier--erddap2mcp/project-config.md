---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
## **🌊 CRITICAL REFERENCE GUIDE 🌊**

**ALWAYS refer to the complete Remote MCP Bible when working on MCP code:**
📖 `/Users/rdc/src/oceancoda/shardrunner-api/docs/remote-mcp-bible.md`

This comprehensive technical guide contains ALL the essential knowledge for building Remote MCP servers, including:
- Complete protocol specifications and examples
- SSE formatting requirements 
- ngrok testing workflows
- Deployment strategies
- Troubleshooting guides
- Security best practices

**REMEMBER: This bible contains the definitive implementation patterns learned through extensive development and testing. Use it as your primary reference for all MCP work.**

## Project Overview

This repository contains **TWO complete ERDDAP MCP servers** that provide tools for searching and accessing ERDDAP (Environmental Research Division's Data Access Program) oceanographic datasets:

### 1. Local MCP Server (`erddapy_mcp_server.py`)
- **Traditional stdio-based MCP server** for local Claude Desktop use
- **4 comprehensive ERDDAP tools** for data discovery and access
- **Config file setup** via `claude_desktop_config.json`

### 2. Remote MCP Server (`erddap_remote_mcp_oauth.py`) 
- **HTTP-based MCP server** for cloud deployment and remote access
- **mcp-remote proxy compatible** for Claude Desktop integration
- **Production-ready** with fly.io deployment configuration
- **Same 4 tools** optimized for remote performance

## 🚨 CRITICAL Remote MCP Knowledge

**HARD-WON TRUTH:** Claude Desktop does NOT support direct remote MCP connections! You MUST use the `mcp-remote` proxy.

### The Secret Architecture That Actually Works:
```
Claude Desktop (stdio) ↔ mcp-remote proxy ↔ Remote MCP Server (HTTP)
```

**Configuration Examples:**

**Local Server:**
```json
{
  "mcpServers": {
    "erddap-local": {
      "command": "python",
      "args": ["/Users/rdc/src/mcp/erddap2mcp/erddapy_mcp_server.py"]
    }
  }
}
```

**Remote Server:**
```json
{
  "mcpServers": {
    "erddap-remote": {
      "command": "npx",
      "args": ["mcp-remote", "https://erddap2mcp.fly.dev/"]
    }
  }
}
```

## Architecture Details

### Local Server Architecture
- **MCP Library**: Uses official `mcp` Python library
- **Communication**: stdio (standard input/output)
- **ERDDAP Integration**: Official `erddapy` client library
- **Tools**: 4 comprehensive ERDDAP access tools
- **Data Processing**: Full pandas DataFrame integration

### Remote Server Architecture  
- **FastAPI Framework**: HTTP server with JSON-RPC 2.0
- **Transport**: StreamableHttp via mcp-remote proxy
- **Protocol Version**: `2025-06-18` (matches Claude Desktop)
- **Tools**: Same 4 ERDDAP tools as local server
- **Deployment**: Containerized with Docker + fly.io

## Development Commands

### Local Server Development
```bash
# Install all dependencies from requirements.txt
pip install -r requirements.txt

# Run local server
python erddapy_mcp_server.py

# Run integration tests
python test_mcp_integration.py
```

### Remote Server Development  
```bash
# Install dependencies
pip install -r requirements.txt

# Run remote server locally
python erddap_remote_mcp_oauth.py
# Server runs on http://localhost:8000

# Test with curl
curl -X POST http://localhost:8000/ \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc": "2.0", "method": "tools/list", "id": 1}'

# Test with mcp-remote proxy
npx mcp-remote http://localhost:8000/ --test
```

### Cloud Deployment (fly.io)
```bash
# Install fly CLI (if not already installed)
curl -L https://fly.io/install.sh | sh

# Login to fly.io
fly auth login

# Deploy to production
fly deploy

# Monitor deployment logs
fly logs -a erddap2mcp

# Check deployment status
fly status -a erddap2mcp

# Server will be available at: https://erddap2mcp.fly.dev/
```

### Docker Build and Run
```bash
# Build Docker image
docker build -t erddap-mcp-server .

# Run container locally
docker run -p 8000:8000 erddap-mcp-server
```

## Available Tools (Both Servers)

1. **list_servers**: Lists ERDDAP servers from `erddaps.json` (63+ servers)
2. **search_datasets**: Search for datasets by keyword
3. **get_dataset_info**: Get detailed metadata about a dataset
4. **to_pandas**: Download and preview data as pandas DataFrame

## Server List Management

**Dynamic Loading from `erddaps.json`:**
- Servers are NO LONGER hardcoded in Python files
- `load_erddap_servers()` function reads from JSON file
- 63 pre-configured servers with worldwide coverage
- Fallback to 2-server minimum if file missing
- JSON structure: name, short_name, url, public flag

## Important Parameters

Both servers accept these common parameters:
- `server_url`: ERDDAP server URL (defaults to NOAA CoastWatch)
- `protocol`: Either "tabledap" (tabular data) or "griddap" (gridded data)
- `dataset_id`: The dataset identifier
- `variables`: List of variables to retrieve
- `constraints`: Dictionary of constraints (e.g., time/space bounds)

## Remote MCP Protocol Implementation

### Key Protocol Requirements
- **JSON-RPC 2.0**: Standard request/response format
- **Protocol Version**: Must be `"2025-06-18"` to match Claude Desktop

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robertdcurrier/erddap2mcp](https://github.com/robertdcurrier/erddap2mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
