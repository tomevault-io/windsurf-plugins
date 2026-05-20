---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

BSV MCP is a Model Context Protocol server that exposes Bitcoin SV (BSV) blockchain functionality to AI assistants. It provides tools for wallet operations, ordinals (NFTs), tokens, identity management, and social features through a modular architecture.

**Current Version: 0.2.10**

The project supports three deployment modes:
1. **Local MCP Server** - Runs via stdio transport with OAuth 2.1 authentication
2. **HTTP Server (Streamable HTTP)** - Runs as HTTP server using MCP 2025-03-26 Streamable HTTP spec with JWT validation
3. **CloudFlare Worker** - Hosted implementation with OAuth 2.1 and Bitcoin-auth

## Essential Commands

### Local Development
```bash
# Install dependencies
bun install

# Build the project
bun build ./index.ts --outdir ./dist --target node

# Run locally (stdio mode for Claude Code)
bun run index.ts

# Run with environment variables
TRANSPORT=stdio USE_DROPLET_API=true bun run index.ts

# Run tests
bun test

# Lint code
bun run lint

# Fix linting issues
bun run lint:fix
```

### CloudFlare Worker Deployment
```bash
# Navigate to cloudflare directory
cd cloudflare

# Install dependencies
bun install

# Deploy the worker
wrangler deploy

# Check worker logs
wrangler tail

# Test locally
wrangler dev
```

### Testing with Claude Code CLI
```bash
# Install as plugin (simplest)
claude plugin install bsv-mcp@b-open-io

# Or add manually via MCP CLI
claude mcp add bsv-mcp "bun run index.ts"

# List configured servers
claude mcp list

# Remove and re-add (useful after changes)
claude mcp remove bsv-mcp
claude mcp add bsv-mcp "bun run index.ts"
```

### CloudFlare Frontend (Next.js)
```bash
cd cloudflare/frontend

# Development
bun dev

# Build
bun run build

# Lint
bun run lint
```

## Core Architecture

### Entry Point
- **index.ts**: Main server initialization with stdio or Streamable HTTP transport
  - Key initialization with SecureKeyManager
  - Transport mode detection (stdio/http)
  - Conditional tool/prompt/resource registration based on env vars
  - `createConfiguredServer()` factory produces a fully-wired `McpServer` instance
  - HTTP mode: one `McpServer` + `WebStandardStreamableHTTPServerTransport` per session (session-per-request model); sessions tracked by `mcp-session-id` header
  - Single `/mcp` endpoint handles all MCP traffic (GET, POST, DELETE); OAuth discovery at `/.well-known/oauth-protected-resource` and `/.well-known/oauth-authorization-server`

### Tool System
- **tools/**: Modular tool categories, each with registration function
  - `bsv/` - Price, transaction decoding, blockchain explorer
  - `wallet/` - Send, receive, UTXOs, ordinals creation, collection minting
  - `ordinals/` - NFT operations, marketplace listings
  - `mnee/` - MNEE token operations
  - `bap/` - Bitcoin Attestation Protocol identity management
  - `bsocial/` - Social posts, likes, follows
  - `bigblocks/` - React component registry and code generation
  - `utils/` - Data conversion, encoding utilities
  - `a2b/` - Agent-to-blockchain publishing (disabled by default)

### Wallet Architecture
- **Dual Mode Support**:
  - **Local Wallet Mode**: Uses `Wallet` class with local private keys
  - **Droplit API Mode**: Uses `IntegratedWallet` with remote faucet service (droplit.dev)
- **Key Types**: Payment key (payPk), Identity key (identityPk), BAP master (xprv)
- **Authentication**: OAuth 2.1 with sigma-auth for MCP clients, BSM for Droplit API operations

### Key Management (SecureKeyManager)
- **Encrypted Storage**: `~/.bsv-mcp/keys.bep` (bitcoin-backup format)
- **Legacy Format**: `~/.bsv-mcp/keys.json` (backward compatible)
- **Dynamic Passphrase Prompting**: Web-based secure passphrase entry
- **Auto-migration**: Converts legacy keys to encrypted format
- **Priority Order**:
  1. PRIVATE_KEY_WIF environment variable (payPk only)
  2. Encrypted keys.bep file (with passphrase prompt)
  3. Legacy keys.json file (unencrypted)
  4. Auto-generate new payPk and save

### Content & Resources
- **prompts/**: Educational content about BSV SDK, ordinals, protocols
- **resources/**: BRC specifications, protocol docs, changelog
- **utils/**: Shared utilities (broadcasting, buffer ops, error handling, key management)

## Tool Registration Pattern

Each tool category follows this pattern:

```typescript
// tools/category/index.ts
export function registerCategoryTools(
  server: McpServer,
  config?: CategoryConfig
): void {
  server.addTool({
    name: "category_toolName",
    description: "...",
    inputSchema: zodSchema,
  }, async (params) => {
    // Implementation
  });
}
```

Main registration in `tools/index.ts` conditionally loads categories based on:
- Environment variables (DISABLE_*_TOOLS, ENABLE_*_TOOLS)
- ToolsConfig object passed from index.ts
- Key availability (payPk, identityPk, xprv)

## Environment Variables

### Core Configuration
- `TRANSPORT`: Transport mode (`stdio` for Claude Code, `http` for Streamable HTTP server, default: `http`)
- `PORT`: HTTP server port (default: 3000)
- `PRIVATE_KEY_WIF`: Bitcoin SV payment private key in WIF format (optional)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [b-open-io/bsv-mcp](https://github.com/b-open-io/bsv-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
