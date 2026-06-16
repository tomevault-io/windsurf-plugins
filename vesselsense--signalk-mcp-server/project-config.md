---
trigger: always_on
description: This file contains instructions and context for Claude when working on this project.
---

# Claude Assistant Instructions

This file contains instructions and context for Claude when working on this project.

SignalK MCP Server - Project Scope & Configuration

## PROJECT OVERVIEW
MCP (Model Context Protocol) server that provides AI agents with access to SignalK marine data via **code execution in V8 isolates**. This approach achieves **90-96% token savings** compared to traditional MCP tools by allowing agents to filter and transform data before returning results.

## CORE FUNCTIONALITY
- Execute JavaScript code in secure V8 isolates with SignalK SDK access
- Connect to SignalK server via HTTP (WebSocket preserved for future streaming)
- Provide current vessel data with client-side filtering
- List nearby AIS targets with distance filtering
- Access system alarms and notifications
- Get latest value for any specific SignalK path
- Monitor connection status to SignalK server
- Discover available data paths on the SignalK installation

## TECHNICAL REQUIREMENTS
Runtime: Node.js 18+ (native fetch support)
Transport: HTTP REST API to SignalK server
Protocol: MCP (Model Context Protocol)
Data Format: JSON (SignalK data)
Architecture: V8 isolate-based code execution with RPC bindings
Dependencies: isolated-vm for secure code execution

## EXPLICITLY OUT OF SCOPE (MVP)
- NO collision avoidance calculations
- NO route planning or navigation
- NO chart data integration
- NO device control functions (read-only only)
- NO complex analytics or historical data
- NO multi-vessel fleet management
- NO weather routing
- NO anchor watch features

## ENVIRONMENT VARIABLES

### SignalK Connection
SIGNALK_HOST=localhost                       # SignalK server hostname/IP (default: localhost)
SIGNALK_PORT=3000                            # SignalK server port (default: 3000)
SIGNALK_TLS=false                            # Use secure connections WSS/HTTPS (default: false)

### Authentication & Context
SIGNALK_TOKEN=                               # Optional authentication token
SIGNALK_CONTEXT=vessels.self                 # Default vessel context

### Execution Mode
EXECUTION_MODE=code                          # code (default) | tools | hybrid
                                             # code: V8 isolate execution only (recommended)
                                             # tools: Legacy MCP tools (deprecated)
                                             # hybrid: Both available (migration mode)

### MCP Server Settings
SERVER_NAME=signalk-mcp-server               # MCP server identifier
SERVER_VERSION=1.0.6                         # Version string
DEBUG=false                                  # Enable debug logging
LOG_LEVEL=info                               # Logging level

### Configuration Examples:
# Local development (default)
SIGNALK_HOST=localhost
SIGNALK_PORT=3000
SIGNALK_TLS=false
EXECUTION_MODE=code

# Legacy tools mode (backward compatibility)
EXECUTION_MODE=tools

## MCP TOOLS

### Primary Tool: execute_code (Code Execution Mode)
Execute JavaScript code in a secure V8 isolate with access to SignalK SDK functions.

**SDK Functions Available (all async - must use `await`):**
- `await getVesselState()` - Get vessel navigation data, position, identity
- `await getAisTargets(options?)` - Get nearby AIS vessels (options: page, pageSize, maxDistance)
- `await getActiveAlarms()` - Get system notifications and alerts
- `await listAvailablePaths()` - Discover available SignalK data paths
- `await getPathValue(path)` - Get value for specific path (accepts string or {path: string})
- `await getConnectionStatus()` - Get connection status

**Code Requirements:**
- Wrap code in async IIFE: `(async () => { ... })()`
- **All SDK functions are async and must be awaited**
- Return `JSON.stringify()` of result object
- Console.log output captured in `logs` array

**Example:**
```javascript
(async () => {
  const vessel = await getVesselState();
  const ais = await getAisTargets({ pageSize: 50 });

  // Filter in isolate - massive token savings!
  const nearby = ais.targets.filter(t => t.distanceMeters < 1852);

  return JSON.stringify({
    vesselName: vessel.data.name?.value,
    position: vessel.data["navigation.position"]?.value,
    nearbyCount: nearby.length
  });
})()
```

### Utility Tools (Available in All Modes)
- `get_connection_status` - Connection health for debugging
- `get_initial_context` - SignalK documentation and context

### Legacy Tools (Deprecated - Use execute_code Instead)
Available only in `tools` or `hybrid` mode:
- `get_vessel_state` -> Use `getVesselState()` in execute_code
- `get_ais_targets` -> Use `getAisTargets()` in execute_code
- `get_active_alarms` -> Use `getActiveAlarms()` in execute_code
- `list_available_paths` -> Use `listAvailablePaths()` in execute_code
- `get_path_value` -> Use `getPathValue()` in execute_code

## TOKEN EFFICIENCY

| Operation | Legacy Tools | Code Execution | Savings |
|-----------|-------------|----------------|---------|
| Vessel state | ~2,000 tokens | ~120 tokens | 94% |
| AIS targets | ~10,000 tokens | ~500 tokens | 95% |
| Active alarms | ~1,000 tokens | ~100 tokens | 90% |
| Available paths | ~2,500 tokens | ~200 tokens | 92% |
| Multi-call workflow | ~13,000 tokens | ~300 tokens | 97% |

## MCP RESOURCES AVAILABLE

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VesselSense/signalk-mcp-server](https://github.com/VesselSense/signalk-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
