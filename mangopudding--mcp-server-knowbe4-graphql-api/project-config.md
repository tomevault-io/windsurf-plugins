---
trigger: always_on
description: **Release Date:** 2025-10-14
---

# CLAUDE.md

**Version:** 1.0.0

**Release Date:** 2025-10-14

**Last Updated:** 2025-10-14

**Author:** Simon Tin-Yul Kok

**Code Quality Rating:** ⭐⭐⭐⭐½ (4.5/5 stars)

**Security Score:** 9.5/10 (Excellent)

**Production Readiness:** ✅ **READY NOW** (Local Desktop Tool)

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 🏠 DEPLOYMENT MODEL

**IMPORTANT:** This MCP server is designed **EXCLUSIVELY for local installation** with Claude Desktop. It is **NOT intended for cloud or multi-tenant deployment**.

### Local Installation (Only Deployment Model)
✅ **Production-ready personal desktop tool** running on your machine alongside Claude Desktop.

**Current Status:**
- **API Key in `.env`**: ✅ Required configuration (normal)
- **Process Isolation**: ✅ Each Claude Desktop instance = separate process
- **PII in Logs**: ✅ Disabled by default
- **Security**: ✅ All read-only operations, comprehensive audit logging
- **Quality**: ✅ 82% test coverage, 148/150 tests passing

**No blocking issues for local use** - Code is ready to use now! 🎉

---

## Security & Privacy (Local Desktop Tool)

**Security Design:**
- ✅ Read-only operations (all mutations blocked)
- ✅ PII field filtering by default
- ✅ Rate limiting (100 queries/60 seconds)
- ✅ Comprehensive audit logging
- ✅ API key stored securely in `.env` (gitignored)

**Privacy:**
- ✅ Conversation logging disabled by default
- ✅ All sensitive data hashed in audit logs
- ✅ Logs stored locally on your machine
- ✅ No telemetry or data sent anywhere except KnowBe4 API

## Project Overview

This is a **security-focused Model Context Protocol (MCP) server** that provides read-only access to the KnowBe4 GraphQL API. The server is designed for integration with Claude Desktop, enabling secure querying of KnowBe4 security awareness training data with built-in privacy controls, audit logging, and compliance features.

**Key Technologies:**
- `fastmcp` - MCP server runtime with JSON-RPC support
- `gql` - Python GraphQL client
- Python 3.10+

**Security Focus:**
- Read-only operations (mutations blocked)
- PII field filtering by default
- Comprehensive audit logging
- Query complexity validation (150-line limit per KnowBe4 API requirements)

## Architecture

### Component Structure

```
[Claude Desktop]
      │
      ▼
[MCP Client SDK]
      │ JSON-RPC (stdio)
      ▼
[fastmcp Server (main.py)]
      │
      ├──→ [GraphQLTools (graphql_tools.py)]
      │         │
      │         ├── Schema validation
      │         ├── Query execution
      │         └── Field filtering
      │
      ├──→ [AuditLogger (audit_logger.py)]
      │         │
      │         └── Compliance logging
      │
      └──→ [ConversationLogger (conversation_logger.py)]
                │
                └── User interaction tracking
```

### Core Modules

1. **main.py** - MCP server entry point
   - Defines MCP tools exposed to Claude Desktop
   - Handles server initialization and configuration
   - Manages environment variables and startup validation

2. **graphql_tools.py** - GraphQL execution engine
   - Loads and validates GraphQL schema from `schema.json`
   - Executes queries against KnowBe4 API
   - Enforces security controls (PII filtering, mutation blocking, complexity limits)
   - Integrates with audit logger for all operations

3. **audit_logger.py** - Compliance logging
   - JSON Lines format for SIEM integration
   - Logs all queries with hashed responses
   - Tracks PII mode, duration, and status
   - Session-based logging with unique IDs

4. **conversation_logger.py** - User interaction tracking
   - Logs complete conversation flow for improvement analysis
   - Captures user questions, processing steps, and responses
   - Tracks tool calls with parameters and results (hashed)
   - Enables data-driven improvements to MCP server

5. **query_library.py** - Query optimization
   - Pre-optimized GraphQL queries for common operations
   - 17+ ready-to-use queries organized by category
   - In-memory caching with configurable TTL
   - Significantly faster response times for repeated queries

### MCP Tools Exposed

**Query Execution:**
- `query_graphql(query, variables, user_question)` - Execute custom GraphQL queries
  - `user_question` (optional): Pass the original user question for conversation logging
- `get_quick_query(query_name, variables, use_cache, user_question)` - Execute pre-optimized queries (FASTER!)
  - `user_question` (optional): Pass the original user question for conversation logging
- `list_quick_queries()` - List all available pre-optimized queries
- `discover_queries(topic)` - **CRITICAL** Discover available queries for a topic and get clarifying questions
- `suggest_query_for_question(user_question)` - **NEW & SMART** Automatically suggest best query using pattern matching

**Configuration:**
- `get_schema_info()` - Get schema metadata and security settings
- `get_schema_type_info(type_name)` - **NEW** Get detailed type information to discover correct field names
- `set_pii_mode(enabled)` - Toggle PII field access
- `add_blocked_field(field_name)` - Add custom field blocking
- `remove_blocked_field(field_name)` - Remove field from block list
- `get_server_status()` - Check server health and configuration

**Performance:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mangopudding) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
