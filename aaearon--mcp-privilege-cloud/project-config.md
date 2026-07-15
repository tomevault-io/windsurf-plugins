---
trigger: always_on
description: **CRITICAL FOR LLM**: This document provides essential development context specifically optimized for AI assistant code development. Read this document completely before making any code changes.
---

# CyberArk Privilege Cloud MCP Server - AI Development Context

**CRITICAL FOR LLM**: This document provides essential development context specifically optimized for AI assistant code development. Read this document completely before making any code changes.

## 🤖 LLM Development Guidelines

**BEFORE CODING**:
1. **Always read this entire CLAUDE.md file first** - Contains critical patterns and constraints
2. **Check current test status** - All changes must maintain 260+ passing tests
3. **Follow existing patterns** - Simplified architecture patterns are established and documented
4. **Use official SDK** - All CyberArk operations MUST use ark-sdk-python (never direct HTTP)
5. **MANDATORY: Use context7 MCP tools for ALL API documentation** - Before working with any library or API, use context7 MCP server tools to get up-to-date documentation

**CODING CONSTRAINTS**:
- ❌ **Never break existing patterns** - Error handling decorator, tool execution, service initialization
- ❌ **Never add boilerplate** - Simplified architecture eliminates repetitive code
- ❌ **Never bypass SDK** - Direct HTTP requests to CyberArk APIs forbidden
- ❌ **Never use outdated documentation** - Always use context7 MCP tools for current API/library docs
- ✅ **Always preserve test coverage** - Every change verified by existing test suite
- ✅ **Always use type hints** - Maintain existing type annotation patterns
- ✅ **Always follow TDD** - Write failing tests first
- ✅ **MANDATORY: Use context7 MCP tools** - Get up-to-date documentation for ANY library/API before coding

## 📚 **MANDATORY: Context7 Documentation Usage**

**🤖 CRITICAL FOR ALL LLM DEVELOPMENT**: Before working with ANY library or API, you MUST use context7 MCP server tools to get current documentation.

### Required Context7 Usage Patterns

**Before working with ark-sdk-python:**
```
Use context7 MCP tools to get latest ark-sdk-python documentation for:
- ArkPCloudAccountsService methods and parameters
- ArkPCloudSafesService API updates  
- ArkPCloudPlatformsService changes
- ArkPCloudApplicationsService methods and operations
- ArkSessionMonitoringService for session monitoring and analytics
- Authentication patterns and model classes

Workflow: resolve-library-id → get-library-docs
```

**Before working with FastMCP:**
```
Use context7 MCP tools to get latest FastMCP documentation for:
- @mcp.tool() decorator updates
- Parameter validation patterns
- Response formatting changes
- Error handling best practices

Workflow: resolve-library-id → get-library-docs
```

**Before working with any Python library:**
```
Use context7 MCP tools to get current documentation for:
- asyncio patterns and best practices
- aiohttp client usage
- pytest testing frameworks
- Type hints and annotation updates

Workflow: resolve-library-id → get-library-docs
```

### Context7 MCP Tool Usage for This Project

```
# Use context7 MCP tools to get library documentation
# Access through your MCP client (Claude Desktop, etc.)

Use context7 resolve-library-id and get-library-docs tools:

1. resolve-library-id: "ark-sdk-python" 
   → get-library-docs with resolved ID

2. resolve-library-id: "fastmcp"
   → get-library-docs with resolved ID  

3. resolve-library-id: "pytest"
   → get-library-docs with resolved ID

4. resolve-library-id: "aiohttp"
   → get-library-docs with resolved ID
```

**🚨 WARNING**: Using outdated API documentation can lead to:
- Deprecated method usage
- Incorrect parameter passing
- Security vulnerabilities
- Test failures
- Integration issues

**✅ RULE**: Always use context7 MCP tools FIRST, then code using current patterns.

## Project Overview

**Purpose**: MCP server for CyberArk Privilege Cloud integration, enabling AI assistants to securely manage privileged accounts.

**Current Status**: ✅ **SERVICE ACCOUNT TOKEN BRIDGE COMPLETE** - OAuth mode verifies user identity via OIDC JWT, then uses a shared service account platform token for all PCloud API calls.
**Last Updated**: February 27, 2026
**Recent Achievement**: Technical debt cleanup — removed dead code (session_manager.py, token_auth.py, HTTP bypasses, unused server methods), simplified AppContext to use `is_oauth` flag, fixed token_verifier exception handling, updated all documentation. 260 passing tests with zero regression.

## Architecture

For a complete overview of the system architecture, see [ARCHITECTURE.md](ARCHITECTURE.md).

## Available MCP Tools

The server provides 53 MCP tools for comprehensive CyberArk operations, built on the official ark-sdk-python library with complete coverage of all 5 PCloud services. For detailed specifications, parameters, examples, and integration patterns, see [API Reference](docs/API_REFERENCE.md).

**Tool Categories**:  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aaearon/mcp-privilege-cloud](https://github.com/aaearon/mcp-privilege-cloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
