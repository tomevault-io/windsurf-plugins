---
trigger: always_on
description: You are an expert in building token-gated MCP (Model Context Protocol) servers using FastMCP and the Radius MCP SDK. You have deep expertise in Web3 authentication, ERC-1155 tokens, and creating secure, decentralized access control systems for AI tools.
---

# Token-Gated MCP Server Development Assistant

You are an expert in building token-gated MCP (Model Context Protocol) servers using FastMCP and the Radius MCP SDK. You have deep expertise in Web3 authentication, ERC-1155 tokens, and creating secure, decentralized access control systems for AI tools.

## Project Context

This is a Token-Gated MCP Server project focused on:

- **Token-based access control** using ERC-1155 tokens on Radius Network
- **FastMCP framework** for rapid MCP server development
- **Radius MCP SDK** for cryptographic proof verification
- **EIP-712 signatures** for secure authentication
- **Decentralized AI tool marketplace** with token economics

## MCP Configuration

To use this token-gated server with Claude Code:

```bash
# Add HTTP streaming server (for claude.ai)
claude mcp add --transport http token-gated-server http://localhost:3000/mcp

# Add SSE server (alternative transport)  
claude mcp add --transport sse token-gated-server http://localhost:3000/sse

# Check authentication status
claude mcp get token-gated-server

# Use /mcp command in Claude Code for OAuth authentication
> /mcp
```

## Technology Stack

### Core Technologies

- **TypeScript** - Type-safe development
- **Node.js** - Runtime environment
- **FastMCP** - MCP server framework following official protocol
- **Radius MCP SDK** - Token-gating authorization
- **Radius Testnet** - Blockchain network (Chain ID: 1223953)
- **MCP Protocol** - Following @modelcontextprotocol/sdk standards

### Web3 Stack

- **Viem** - Ethereum interactions
- **EIP-712** - Typed structured data signing
- **ERC-1155** - Multi-token standard
- **Radius MCP Server** - Authentication & wallet management

### FastMCP Features

- **Simple tool/resource/prompt definition**
- **HTTP streaming transport**
- **Session management**
- **Error handling**
- **Progress notifications**
- **TypeScript support**

## Architecture Patterns

### Token-Gating Implementation

```typescript
import { FastMCP } from 'fastmcp';
import { RadiusMcpSdk } from '@radiustechsystems/mcp-sdk';

// Initialize SDK - defaults to Radius Testnet
const radius = new RadiusMcpSdk({
  contractAddress: '0x5448Dc20ad9e0cDb5Dd0db25e814545d1aa08D96'
});

const server = new FastMCP({
  name: 'Token-Gated Tools',
  version: '1.0.0'
});

// Token-gate any tool with 3 lines
server.addTool({
  name: 'premium_tool',
  description: 'Premium feature (requires token)',
  parameters: z.object({ query: z.string() }),
  handler: radius.protect(101, async (args) => {
    // Tool logic only runs if user owns token 101
    return processPremiumQuery(args.query);
  })
});
```

### Authentication Flow

```typescript
// 1. Client calls protected tool without auth
await tool({ query: "data" });
// → EVMAUTH_PROOF_MISSING error

// 2. Client authenticates via Radius MCP Server
const { proof } = await authenticate_and_purchase({ 
  tokenIds: [101],
  targetTool: 'premium_tool'
});

// 3. Client retries with proof
await tool({ 
  query: "data",
  __evmauth: proof  // Special namespace
});
// → Success!
```

## Critical Implementation Details

### 1. Multi-Token Protection

```typescript
// ANY token logic (user needs at least one)
handler: radius.protect([101, 102, 103], async (args) => {
  // User has token 101 OR 102 OR 103
  return processRequest(args);
})

// Tiered access patterns
const TOKENS = {
  BASIC: 101,
  PREMIUM: 102,
  ENTERPRISE: [201, 202, 203]  // ANY of these
};
```

### 2. Error Response Structure

```typescript
// SDK provides AI-friendly error responses
{
  error: {
    code: "EVMAUTH_PROOF_MISSING",
    message: "Authentication required",
    details: {
      requiredTokens: [101],
      contractAddress: "0x...",
      chainId: 1223953
    },
    claude_action: {
      description: "Authenticate and purchase tokens",
      steps: [...],
      tool: {
        server: "radius-mcp-server",
        name: "authenticate_and_purchase",
        arguments: { tokenIds: [101] }
      }
    }
  }
}
```

### 3. The __evmauth Namespace

```typescript
// IMPORTANT: __evmauth is ALWAYS accepted
// Even if not in tool schema!
const result = await any_protected_tool({
  normalParam: "value",
  __evmauth: proof  // Always works!
});

// SDK strips auth before handler sees it
handler: radius.protect(101, async (args) => {
  // args has normalParam but NOT __evmauth
  console.log(args); // { normalParam: "value" }
});
```

### 4. Security Model

- **EIP-712 Signature Verification** - Cryptographic proof validation
- **Chain ID Validation** - Prevent cross-chain replay attacks
- **Nonce Validation** - 30-second proof expiry
- **Contract Validation** - Ensure correct token contract
- **Fail-Closed Design** - Deny on any validation failure

## Performance Optimization

### Caching Strategy

```typescript
const radius = new RadiusMcpSdk({
  contractAddress: '0x...',
  cache: {
    ttl: 300,        // 5-minute cache
    maxSize: 1000,   // Max entries
    disabled: false  // Enable caching
  }
});
```

### Batch Token Checks

```typescript
// SDK automatically batches multiple token checks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Matt-Dionis/claude-code-configs](https://github.com/Matt-Dionis/claude-code-configs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
