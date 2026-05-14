---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**n8n-manager-for-ai-agents** - An MCP server that enables Claude Desktop to manage n8n workflow automation instances through the n8n API.

### n8n Compatibility Requirements
- **Minimum n8n version**: 1.0.0
- **API version**: v1
- **Feature detection**: Runtime checks for enterprise features

### Key Features
- Complete n8n API coverage through MCP tools (within API constraints)
- Workflow creation, management, and webhook-based execution
- Credential management with secure handling
- Tag and variable management
- Audit and security reporting
- Multi-instance support

### Known n8n API Limitations
- **No direct workflow execution**: Must use webhook triggers
- **No execution control**: Cannot stop running executions
- **No user management**: Not exposed via public API
- **No credential schemas**: Cannot retrieve credential type definitions
- **Limited filtering**: Fewer options than expected
- **Cursor-based pagination**: Not offset-based as commonly expected
- **Settings field required**: Workflow creation now requires 'settings' object (not documented)
- **PATCH method restrictions**: Some instances don't support PATCH for workflow updates (use PUT instead)

### Implemented Workarounds
1. **Webhook-Based Execution**: Workflows must have webhook triggers for execution
2. **Polling for Results**: Check execution status periodically
3. **Client-Side Filtering**: Additional filtering done in MCP server
4. **Feature Detection**: Runtime checks for available endpoints
5. **Variables via Source Control**: Using source control API for variable management
6. **Default Settings**: Automatically includes required settings with sensible defaults
7. **Method Fallback**: Tries PUT method first, falls back to PATCH for compatibility
8. **Full Workflow Updates**: Fetches existing workflow data for partial updates when needed

## Quick Reference: What Works vs What Doesn't

### ✅ Available via API
- Workflow CRUD operations
- Listing and getting executions (but NOT stopping them)
- Credential management (without schema info)
- Tags management
- Import/export workflows
- Source control operations
- Health checks

### ❌ NOT Available via API (Don't implement)
- Direct workflow execution (use webhooks instead)
- Stopping running executions
- User management (all CRUD operations)
- Credential type schemas
- Execution date filtering
- Community nodes management

### Technology Stack
- **TypeScript**: Primary language
- **MCP SDK**: @modelcontextprotocol/sdk v1.13.1
- **HTTP Client**: Axios for n8n API communication
- **Validation**: Zod for schema validation
- **Logging**: Winston for structured logging
- **Testing**: Jest with TypeScript support

## Commands

### Initial Setup
```bash
# Install dependencies
npm install

# Build TypeScript
npm run build

# Run tests
npm test
npm run test:integration

# Start MCP server
npm start
```

### Development
```bash
# Type checking
npm run typecheck

# Linting
npm run lint

# Watch mode for development
npm run dev

# Run with debug logging
LOG_LEVEL=debug npm start

# Test with MCP Inspector
npx @modelcontextprotocol/inspector build/index.js
```

## Architecture

### System Design
```
Claude Desktop <-> MCP Protocol <-> n8n MCP Server <-> n8n API <-> n8n Instance
```

### Core Components

1. **MCP Server** (`src/server.ts`)
   - Implements MCP protocol using @modelcontextprotocol/sdk v1.13.1
   - Registers all n8n API operations as tools
   - Handles stdio transport for Claude Desktop

2. **n8n API Client** (`src/services/n8nClient.ts`)
   - Axios-based HTTP client
   - API key authentication (X-N8N-API-KEY header)
   - Connection pooling and retry logic

3. **Tool Implementations** (`src/tools/`)
   - `workflows.ts` - Workflow CRUD operations
   - `executions.ts` - Execution management
   - `credentials.ts` - Credential handling
   - `users.ts` - User management (Enterprise)
   - `audit.ts` - Security audit tools

4. **Services** (`src/services/`)
   - Input validation with Zod schemas
   - Error handling and retry logic
   - Caching for performance

### Project Structure
```
src/
├── index.ts              # Entry point
├── server.ts             # MCP server class
├── config/
│   └── environment.ts    # Environment config
├── tools/
│   ├── workflows.ts      # Workflow tools
│   ├── executions.ts     # Execution tools
│   ├── credentials.ts    # Credential tools
│   ├── users.ts          # User tools
│   └── audit.ts          # Audit tools
├── services/
│   ├── n8nClient.ts      # n8n API client
│   ├── validation.ts     # Input validation
│   └── cache.ts          # Caching service
├── utils/
│   ├── logger.ts         # Winston logger
│   └── errors.ts         # Error handling
└── types/
    └── index.ts          # TypeScript types
```

## MCP Tools Reference

### Workflow Management
- `n8n_create_workflow` - Create new workflow
- `n8n_get_workflow` - Get workflow by ID
- `n8n_update_workflow` - Update workflow
- `n8n_delete_workflow` - Delete workflow
- `n8n_list_workflows` - List workflows with filters
- `n8n_activate_workflow` - Activate workflow

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [czlonkowski/n8n-manager-for-ai-agents](https://github.com/czlonkowski/n8n-manager-for-ai-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
