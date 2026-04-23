---
trigger: always_on
description: Generates AgentCard and manifest JSON. Includes A2A skills, payments metadata, trust registrations.
---

# Lucid Agents Monorepo - AI Coding Guide

This guide helps AI coding agents understand and work with the lucid-agents monorepo effectively.

## Project Overview

This is a TypeScript/Bun monorepo for building, monetizing, and verifying AI agents. It provides:

- **@lucid-agents/core** - Protocol-agnostic agent runtime with extension system
- **@lucid-agents/http** - HTTP extension for request/response handling
- **@lucid-agents/identity** - ERC-8004 identity and trust layer
- **@lucid-agents/payments** - x402 payment utilities
- **@lucid-agents/wallet** - Wallet SDK for agent and developer wallets
- **@lucid-agents/a2a** - A2A Protocol client for agent-to-agent communication
- **@lucid-agents/ap2** - AP2 (Agent Payments Protocol) extension
- **@lucid-agents/hono** - Hono HTTP server adapter
- **@lucid-agents/express** - Express HTTP server adapter
- **@lucid-agents/tanstack** - TanStack Start adapter
- **@lucid-agents/cli** - CLI for scaffolding new agent projects

**Tech Stack:**

- Runtime: Bun (Node.js 20+ compatible)
- Language: TypeScript (ESM, strict mode)
- Build: tsup
- Package Manager: Bun workspaces
- Versioning: Changesets

## Architecture Overview

### Package Dependencies

```
cli (scaffolding tool)
    ↓ scaffolds projects using
hono OR express OR tanstack (adapters)
    ↓ both use
core (protocol-agnostic runtime)
    ↓ uses extensions
http, identity, payments, wallet, a2a, ap2 (extensions)
```

### Extension System

The framework uses an extension-based architecture where features are added via composable extensions:

- **http** (`@lucid-agents/http`) - HTTP request/response handling, streaming, SSE
- **wallets** (`@lucid-agents/wallet`) - Wallet management for agents
- **payments** (`@lucid-agents/payments`) - x402 payment verification and pricing
- **identity** (`@lucid-agents/identity`) - ERC-8004 on-chain identity and trust
- **a2a** (`@lucid-agents/a2a`) - Agent-to-agent communication protocol
- **ap2** (`@lucid-agents/ap2`) - Agent Payments Protocol extension

### Adapter System

The framework supports multiple runtime adapters:

- **Hono** (`@lucid-agents/hono`) - Traditional HTTP server
- **Express** (`@lucid-agents/express`) - Node.js/Express server with x402 middleware
- **TanStack Start** (`@lucid-agents/tanstack`) - Full-stack React with dashboard (UI) or API-only (headless)

Templates are adapter-agnostic and work with any compatible adapter.

### Data Flow

```
HTTP Request
    ↓
Adapter Router (Hono, Express, or TanStack)
    ↓
x402 Paywall Middleware (if configured)
    ↓
Runtime Handler (core)
    ↓
Entrypoint Handler
    ↓
Response (JSON or SSE stream)
```

### Key Architectural Decisions

1. **Multi-adapter support** - Same agent logic works with different frameworks
2. **Template-based scaffolding** - Templates use `.template` extensions for clean code generation
3. **Zod for validation** - Schema-first approach for input/output
4. **Server-Sent Events for streaming** - Standard SSE for real-time responses
5. **ERC-8004 for identity** - On-chain agent identity and reputation
6. **x402 for payments** - HTTP-native payment protocol supporting both EVM and Solana networks

### Supported Payment Networks

The framework supports payment receiving on multiple blockchain networks:

**EVM Networks:**

- `base` - Base mainnet (L2, low cost)
- `base-sepolia` - Base Sepolia testnet
- `ethereum` - Ethereum mainnet
- `sepolia` - Ethereum Sepolia testnet

**Solana Networks:**

- `solana` - Solana mainnet (high throughput, low fees)
- `solana-devnet` - Solana devnet

**Key Differences:**

- **EVM**: EIP-712 signatures, ERC-20 tokens (USDC), 0x-prefixed addresses
- **Solana**: Ed25519 signatures, SPL tokens (USDC), Base58 addresses
- **Transaction finality**: Solana (~400ms) vs EVM (12s-12min)
- **Gas costs**: Solana (~$0.0001) vs EVM ($0.01-$10)

**Identity vs Payments:**

- Identity registration (ERC-8004): EVM-only (smart contract on Ethereum chains)
- Payment receiving: Any supported network (EVM or Solana)
- These are independent: register identity on Base, receive payments on Solana

## Code Structure Principles

These principles guide how we organize and structure code across the monorepo. Follow them when writing new code or refactoring existing code.

### 1. Single Source of Truth

**One type definition per concept.** Avoid duplicate types like `PaymentsRuntimeInternal` vs `PaymentsRuntime`. If you need variations, use type composition or generics, not separate type definitions.

**Bad:**

```typescript
// Internal type
type PaymentsRuntimeInternal = { config: PaymentsConfig | undefined; activate: ... };
// Public type
type PaymentsRuntime = { config: PaymentsConfig; requirements: ... };
```

**Good:**

```typescript
// One type definition
type PaymentsRuntime = {
  config: PaymentsConfig;
  isActive: boolean;
  requirements: ...;
  activate: ...;
};
```

### 2. Encapsulation at the Right Level

**Domain complexity belongs in the owning package.** The payments package should handle all payments-related complexity. The core runtime should use it directly without transformation layers.

**Bad:**

```typescript
// In core runtime - wrapping payments runtime
const paymentsRuntime = payments.config ? {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daydreamsai/lucid-agents](https://github.com/daydreamsai/lucid-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
