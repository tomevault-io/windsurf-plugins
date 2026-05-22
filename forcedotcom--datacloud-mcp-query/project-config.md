---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

<!-- Test change for SSH signing -->

## Project Overview

This is a Model Context Protocol (MCP) server that provides integration between Cursor IDE and Salesforce Data Cloud. It exposes tools for executing SQL queries, listing tables, and describing table structures through the MCP protocol using the FastMCP framework.

## Setup and Development

### Installation

```bash
# Install dependencies
pip install -r requirements.txt
```

### Running the Server

**Option 1: Using SF CLI Authentication (Recommended)**
```bash
# Prerequisites: Authenticate with SF CLI
sf org login web --alias myorg

# Set required environment variable
export SF_ORG_ALIAS="myorg"

# Optional environment variables
export DEFAULT_LIST_TABLE_FILTER="%"  # default, SQL LIKE pattern

# Run the server
python server.py
```

**Option 2: Using OAuth PKCE Authentication**
```bash
# Set required environment variables
export SF_CLIENT_ID="your_client_id"
export SF_CLIENT_SECRET="your_client_secret"

# Optional environment variables
export SF_LOGIN_URL="login.salesforce.com"  # default
export SF_CALLBACK_URL="http://localhost:55556/Callback"  # default
export DEFAULT_LIST_TABLE_FILTER="%"  # default, SQL LIKE pattern

# Run the server
python server.py
```

### Testing Individual Components

```bash
# Test the Data Cloud SQL client directly
python connect_api_dc_sql.py

# This will prompt for OAuth authentication and run a test query
```

## Architecture

### Four-Layer Design

The codebase follows a clean four-layer architecture with pluggable authentication:

1. **MCP Server Layer** (`server.py`)
   - Entry point for the MCP server using FastMCP framework
   - Defines three MCP tools: `query()`, `list_tables()`, `describe_table()`
   - Uses authentication abstraction via `auth_provider` interface
   - All SQL uses PostgreSQL dialect

2. **Authentication Abstraction Layer**
   - `auth_interface.py`: Defines `AuthProvider` protocol with `get_token()`, `get_instance_url()`, and `is_available()` methods
   - `auth_factory.py`: Auto-detection logic that selects appropriate authentication method
   - Implements authentication priority: SF CLI → OAuth → Error with setup instructions

3. **Authentication Implementations**
   - **SF CLI Authentication** (`sf_cli_auth.py`):
     - Uses `sf org display --target-org <alias> --json` subprocess to obtain tokens
     - Caches tokens for 5 minutes to reduce overhead
     - Validates SF CLI availability and org authentication
   - **OAuth PKCE Authentication** (`oauth.py`):
     - `OAuthConfig`: Loads configuration from environment variables
     - `OAuthSession`: Manages access token lifecycle with browser-based flow
     - Implements OAuth2 with PKCE extension for security
     - Auto-refreshes tokens after 110-minute expiration
     - Opens browser for user authentication via local HTTP callback server

4. **Data Cloud API Layer** (`connect_api_dc_sql.py`)
   - `run_query()`: Core query execution against Salesforce Data Cloud Query API
   - Uses Salesforce REST API v63.0 endpoint: `/services/data/v63.0/ssot/query-sql`
   - Handles asynchronous query execution with polling
   - Implements automatic pagination for large result sets (default: 100K rows per batch)
   - Returns structured response: `{"data": [...], "metadata": [...]}`
   - Works with any `AuthProvider` implementation via protocol

### Key Patterns

**Authentication Auto-Detection:**
- Factory pattern with automatic fallback
- Priority: SF CLI (if `SF_ORG_ALIAS` set) → OAuth (if credentials set) → Error
- Validates authentication during initialization with clear error messages
- All authentication providers implement the same `AuthProvider` protocol

**SF CLI Token Management:**
- Tokens cached for 5 minutes (balance between performance and freshness)
- Subprocess timeout: 120 seconds
- Comprehensive error messages guide users to `sf org login web`
- Validates SF CLI JSON output structure before using

**OAuth Token Management:**
- Tokens are lazily initialized on first use
- `ensure_access()` checks expiration and re-authenticates if needed
- Browser-based OAuth flow opens automatically when token is missing/expired
- Uses PKCE (Proof Key for Code Exchange) for enhanced security

**Query Execution Flow:**
1. Submit SQL query via POST (with optional `sqlParameters`) to get `queryId`
2. Poll query status with long-polling (`waitTimeMs=10000`) until completion
3. Retrieve results via pagination if `rowCount > initial rows returned`
4. Aggregate all pages into single response

**Parameterized Queries:**
- `run_query()` accepts an optional `sql_parameters` list for safe value binding
- Parameters use `:paramName` placeholders in SQL, resolved server-side by Data Cloud
- Format: `[{"name": "paramName", "value": "someValue"}]` (optional `"type"` key)
- Used by `describe_table()` to safely pass the table name without string interpolation

**Workload Management:**
- Default workload name: `"data-360-mcp-query-oss"`
- Workload name can be customized via `workload_name` parameter in `run_query()`
- Workloads are used for query resource management in Salesforce Data Cloud


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [forcedotcom/datacloud-mcp-query](https://github.com/forcedotcom/datacloud-mcp-query) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
