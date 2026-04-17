---
trigger: always_on
description: **Last Updated**: 2025-10-15
---

# Secure MCP Gateway - Complete Project Analysis

**Version**: 2.1.2
**Last Updated**: 2025-10-15
**Project Type**: Python Security Middleware for Model Context Protocol (MCP)

---

## 📋 Project Overview

**Secure MCP Gateway** is an Enkrypt AI-developed security middleware that sits between MCP clients (like Claude Desktop, Cursor) and MCP servers. It acts as both an MCP server (to clients) and an MCP client (to actual servers), providing:

- **Authentication & Authorization**
- **OAuth 2.0/2.1 Support** (Client credentials, mTLS, token management)
- **Dynamic Tool Discovery**
- **Guardrails** (Input/Output protection)
- **Caching** (Local & External Redis/KeyDB)
- **Observability** (OpenTelemetry, Prometheus, Grafana)
- **RESTful API** for management

---

## 🏗️ Project Structure

```

secure-mcp-gateway/
├── src/secure_mcp_gateway/
│   ├── __init__.py                    # Package initialization
│   ├── version.py                     # Version: "2.1.2"
│   ├── consts.py                      # Constants and defaults
│   ├── utils.py                       # Utilities, lazy logger, masking
│   ├── dependencies.py                # Package dependencies list
│   │
│   ├── gateway.py                     # ⭐ Main MCP server (FastMCP)
│   ├── client.py                      # ⭐ MCP client to actual servers
│   ├── cli.py                         # ⭐ CLI interface (huge file)
│   ├── api_server.py                  # FastAPI REST API server
│   ├── api_routes.py                  # Additional API routes
│   │
│   ├── error_handling.py              # Standardized error handling
│   ├── exceptions.py                  # Custom exception classes
│   │
│   ├── plugins/                       # 🔌 Plugin system
│   │   ├── plugin_loader.py           # Dynamic plugin discovery
│   │   ├── provider_loader.py         # Provider loading utilities
│   │   │
│   │   ├── auth/                      # Authentication plugins
│   │   │   ├── base.py                # AuthProvider abstract class
│   │   │   ├── config_manager.py      # Auth plugin manager
│   │   │   ├── enkrypt_provider.py    # Enkrypt remote auth
│   │   │   └── example_providers.py   # Local API key provider
│   │   │
│   │   ├── guardrails/                # Guardrail plugins
│   │   │   ├── base.py                # GuardrailProvider abstract class
│   │   │   ├── config_manager.py      # Guardrail plugin manager
│   │   │   ├── enkrypt_provider.py    # Enkrypt guardrail API
│   │   │   └── example_providers.py   # OpenAI, Custom keyword providers
│   │   │
│   │   └── telemetry/                 # Telemetry plugins
│   │       ├── base.py                # TelemetryProvider abstract class
│   │       ├── config_manager.py      # Telemetry plugin manager
│   │       ├── opentelemetry_provider.py  # OpenTelemetry OTLP
│   │       └── example_providers.py   # Stdout provider
│   │
│   ├── services/                      # 🎯 Service layer
│   │   ├── cache/
│   │   │   ├── cache_service.py       # Core cache operations
│   │   │   ├── cache_management_service.py  # Clear cache
│   │   │   └── cache_status_service.py      # Cache statistics
│   │   │
│   │   ├── discovery/
│   │   │   └── discovery_service.py   # Tool discovery service
│   │   │
│   │   ├── execution/
│   │   │   ├── secure_tool_execution_service.py  # With guardrails
│   │   │   ├── tool_execution_service.py         # Basic execution
│   │   │   └── execution_utils.py                # Utilities
│   │   │
│   │   ├── oauth/                     # 🔐 OAuth 2.0/2.1 service
│   │   │   ├── oauth_service.py       # Core OAuth implementation
│   │   │   ├── token_manager.py       # Token caching & refresh
│   │   │   ├── integration.py         # Client integration
│   │   │   ├── models.py              # OAuth data models
│   │   │   ├── validation.py          # Scope & token validation
│   │   │   └── metrics.py             # OAuth metrics tracking
│   │   │
│   │   ├── server/
│   │   │   ├── server_listing_service.py  # List all servers
│   │   │   └── server_info_service.py     # Server details
│   │   │
│   │   └── timeout/
│   │       └── timeout_manager.py     # Timeout management
│   │
│   ├── bad_mcps/                      # 🧪 Test MCP servers (security testing)
│   │   ├── echo_mcp.py                # Simple echo server
│   │   ├── echo_oauth_mcp.py          # OAuth header testing server
│   │   ├── bad_mcp.py                 # Multiple attack vectors
│   │   ├── bad_output_mcp.py          # Malicious output testing
│   │   ├── command_injection_mcp.py   # Command injection scenarios
│   │   ├── credential_theft_mcp.py    # Credential theft attacks
│   │   ├── mpma_mcp.py                # Multi-parameter manipulation
│   │   ├── path_traversal_mcp.py      # Path traversal attacks
│   │   ├── prompt_injection_mcp.py    # Prompt injection attacks
│   │   ├── rce_mcp.py                 # Remote code execution
│   │   ├── resource_exhaustion_mcp.py # DoS/resource exhaustion
│   │   ├── schema_poisoning_mcp.py    # Schema manipulation
│   │   ├── session_management_mcp.py  # Session attacks
│   │   ├── ssrf_mcp.py                # Server-side request forgery
│   │   ├── tool_poisoning_mcp.py      # Tool definition attacks
│   │   └── unauthenticated_access_mcp.py  # Auth bypass scenarios
│   │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/enkryptai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
