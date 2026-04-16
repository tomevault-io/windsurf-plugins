---
trigger: always_on
description: **🔥 This service channels the divine MCP protocol through native StreamableHTTP glory! ⚡**
---

# MCP Fetch StreamableHTTP Server - Divine Implementation Guide

**🔥 This service channels the divine MCP protocol through native StreamableHTTP glory! ⚡**

## Sacred Service Overview

**⚡ Native MCP implementation - No wrapped servers, pure protocol enlightenment! ⚡**

This blessed service implements:
- **Native StreamableHTTP transport** - Direct protocol implementation without stdio bridging!
- **Fetch tool** - Divine URL retrieval with security protections!
- **Stateless operation** - Each request stands alone in holy isolation!
- **FastAPI + Uvicorn** - Modern async Python stack blessed with uvloop performance!

## The Divine Trinity Architecture Compliance

**🔱 This service exists in Layer 3 - The Pure Protocol Servants of Glory! 🔱**

### Sacred Separation of Concerns
- **No authentication code** - Traefik handles all auth via ForwardAuth middleware!
- **No CORS handling** - Traefik manages all cross-origin blessings!
- **No routing logic** - Pure MCP protocol implementation only!
- **No OAuth awareness** - Receives pre-authenticated requests only!

**⚡ This service knows only MCP protocol - authentication is Traefik's divine responsibility! ⚡**

## Sacred Protocol Implementation

### StreamableHTTP Transport Endpoints

**The Holy `/mcp` Endpoint - The Single Gateway to Protocol Paradise!**

```python
POST /mcp     # JSON-RPC request processing
GET /mcp      # SSE endpoint (infrastructure ready, not yet implemented)
DELETE /mcp   # Session termination (infrastructure ready)
```

### Supported MCP Methods

**Divine Protocol Methods Implemented:**
- `initialize` - Protocol handshake and capability negotiation!
- `tools/list` - Returns the blessed fetch tool!
- `tools/call` - Executes fetch operations with divine protection!

**Declared Capabilities:**
```json
{
  "tools": {},       // Full tool support blessed!
  "prompts": null,   // Not implemented (honest declaration!)
  "resources": null, // Not implemented (truthful admission!)
  "logging": null    // Not implemented (sacred transparency!)
}
```

## The Sacred Fetch Tool

**🌐 Divine URL retrieval with security blessings! 🌐**

### Tool Definition
```json
{
  "name": "fetch",
  "description": "Fetch a URL and return its contents",
  "inputSchema": {
    "type": "object",
    "properties": {
      "url": {
        "type": "string",
        "description": "The URL to fetch",
        "format": "uri"
      },
      "method": {
        "type": "string",
        "description": "HTTP method to use",
        "enum": ["GET", "POST"],
        "default": "GET"
      },
      "headers": {
        "type": "object",
        "description": "Optional HTTP headers to send"
      },
      "body": {
        "type": "string",
        "description": "Optional request body (for POST requests)"
      },
      "max_length": {
        "type": "integer",
        "description": "Maximum response length in bytes",
        "default": 100000
      },
      "user_agent": {
        "type": "string",
        "description": "User agent string to use",
        "default": "ModelContextProtocol/1.0"
      }
    },
    "required": ["url"]
  }
}
```

### Divine Security Protections

**⚡ SSRF Protection - Blocks access to internal networks! ⚡**
- Localhost and 127.0.0.0/8 blocked!
- Private networks (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16) forbidden!
  - **Divine note**: Uses prefix matching for simplicity (e.g., "172." blocks all 172.x.x.x)
- AWS metadata service (169.254.169.254) denied!
- Link-local addresses blocked!

**🔒 Content Security:**
- Response size limited (100KB default)!
- Automatic content-type detection!
- Images returned as base64!
- HTML title extraction for text/html!
- Robots.txt compliance when available!

## Sacred Environment Variables

**⚙️ All configuration flows through blessed environment! ⚡**

```bash
# Required Divine Configuration
MCP_SERVER_NAME=mcp-fetch         # Server identity declaration!
MCP_SERVER_VERSION=1.0.0          # Version proclamation!
MCP_PROTOCOL_VERSION=2025-06-18   # Protocol covenant version!

# Optional Blessed Settings
MCP_FETCH_ALLOWED_SCHEMES=["http","https"]  # Allowed URL schemes!
MCP_FETCH_MAX_REDIRECTS=5                   # Redirect limit!
MCP_FETCH_DEFAULT_USER_AGENT=ModelContextProtocol/1.0 (Fetch Server)
MCP_FETCH_ENABLE_SSE=false        # Future SSE support flag!

# Server Configuration
HOST=0.0.0.0                      # Binding address!
PORT=3000                         # Sacred port 3000!
```

**⚡ Never hardcode values - all config through environment or face damnation! ⚡**

## Health Check Pattern

**🏥 StreamableHTTP protocol health verification! 🏥**

The divine health check uses protocol initialization:
```yaml
healthcheck:
  test: ["CMD", "sh", "-c", "curl -s -X POST http://localhost:3000/mcp \
    -H 'Content-Type: application/json' \
    -d '{\"jsonrpc\":\"2.0\",\"method\":\"initialize\",\"params\":{\"protocolVersion\":\"${MCP_PROTOCOL_VERSION:-2025-06-18}\",\"capabilities\":{},\"clientInfo\":{\"name\":\"healthcheck\",\"version\":\"1.0\"}},\"id\":1}' \
    | grep -q '\"protocolVersion\":\"${MCP_PROTOCOL_VERSION:-2025-06-18}\"'"]
  interval: 30s
  timeout: 5s
  retries: 3
  start_period: 40s
```

**⚡ This blessed incantation verifies protocol readiness! ⚡**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/atrawog) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
