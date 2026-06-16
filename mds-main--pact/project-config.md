---
trigger: always_on
description: > **PACT** (Policy-Aware Crypto Transactor) is an autonomous AI agent platform for onchain procurement on Base. AI agents discover counterparties, negotiate deals, escrow USDC, encrypt artifacts, verify deliverables, and settle onchain — all within human-defined mandate policies.
---

# AGENTS.md — PACT Agent Architecture

> **PACT** (Policy-Aware Crypto Transactor) is an autonomous AI agent platform for onchain procurement on Base. AI agents discover counterparties, negotiate deals, escrow USDC, encrypt artifacts, verify deliverables, and settle onchain — all within human-defined mandate policies.

## Agent Identity

| Field | Value |
|-------|-------|
| **Name** | PACT |
| **Type** | Autonomous procurement agent |
| **Chain** | Base Sepolia (84532) |
| **AI Provider** | Venice AI (`zai-org-glm-4.7`) via Vercel AI SDK |
| **Agent Card** | `/.well-known/agent-card.json` |
| **Escrow Contract** | `PACTEscrow.sol` (OpenZeppelin SafeERC20 + ReentrancyGuard) |
| **Wallet** | Lit Protocol PKP (MPC threshold signing) in production; EOA in dev |

## Architecture Overview

```
User (Browser)
  ↓  natural language
Chat API (/api/chat)
  ↓  Vercel AI SDK + Venice AI
Agentic Tools (6 tools)
  ↓  structured actions
Vercel Workflow DevKit (durable pipelines)
  ↓  crash-resilient state machine
Onchain (Base Sepolia)
  ↓  USDC escrow, ERC-8004 identity, ENS resolution
Settlement
```

## Agentic Tools

The agent exposes 6 tools via `lib/ai/tools.ts`, wired into Venice AI through the Vercel AI SDK:

| Tool | Purpose |
|------|---------|
| `createMandate` | Compile natural-language policy into structured JSON (spend caps, privacy, approval thresholds) |
| `listMandates` | List all policy configurations for the user |
| `getActiveMandate` | Get current active policy — enforced on every deal |
| `createDealRequest` | Parse procurement intent via Venice AI, validate budget against mandate |
| `runDealWorkflow` | Start the full deal pipeline (auto-triggered after `createDealRequest`) |
| `getDealStatus` | Check deal progress and current state |

## AI Roles (6 Venice AI Inference Passes)

Each role uses Venice structured outputs for reliable JSON extraction:

1. **Policy Compiler** (`lib/ai/services/compile-policy.ts`) — Natural language → `MandatePolicy` JSON
2. **Intent Parser** (`lib/ai/services/parse-intent.ts`) — Task description → `DealIntent` with budget, deliverables, deadline
3. **Counterparty Scorer** (`lib/ai/services/match-capabilities.ts`) — Semantic capability matching via Venice AI
4. **Negotiator** (`lib/ai/services/negotiate.ts`) — Autonomous deal negotiation with counterparties
5. **Verifier** (`lib/ai/services/verify-delivery.ts`) — Deliverable verification via multimodal analysis
6. **Evidence Writer** (`lib/evidence/builder.ts`) — Generate judge-readable evidence bundles

## Deal State Machine

```solidity
// contracts/PACTEscrow.sol — 10 states
NONE → OPEN → ACCEPTED → DELIVERED → VERIFIED → RELEASED
                                    → APPROVAL_PENDING → RELEASED
       OPEN → EXPIRED → REFUNDED
              DELIVERED → DISPUTED → REFUNDED
```

The smart contract enforces:
- USDC escrow on deal open (`SafeERC20.safeTransferFrom`)
- Counterparty acceptance required
- Delivery hash + URI for content-addressed verification
- Human approval gate (configurable per deal)
- Reentrancy protection on all fund movements

## Protocol Integrations

### ERC-8004 — Agent Identity & Reputation
- **Identity Registry** (`lib/erc8004/registry.ts`): Read agent registrations (`ownerOf`, `tokenURI`, `getAgentWallet`, `getMetadata`)
- **Reputation Registry** (`lib/erc8004/feedback.ts`): Write feedback after settlement (`giveFeedback` with tags: `successRate`, `responseTime`, `starred`)
- **Discovery Source** (`lib/discovery/sources/erc8004.ts`): Discover counterparties from on-chain registry

### ENS — Agent Discovery
- **Resolver** (`lib/ens/resolver.ts`): Read text records (`pact.endpoint`, `pact.capabilities`, `pact.agentWallet`)
- **Discovery Source** (`lib/discovery/sources/ens.ts`): ENS names replace hex addresses for counterparties

### Lit Protocol — Encryption & Agent Wallet
- **PKP Wallet** (`lib/lit/pkp.ts`): MPC threshold signing via `Lit.Actions.signEcdsa` — private key never exists on one machine
- **Encrypt/Decrypt** (`lib/lit/encrypt.ts`, `lib/lit/decrypt.ts`): Symmetric encryption with Lit access conditions
- **Access Conditions** (`lib/lit/access-conditions.ts`): Attribute-based access control for encrypted artifacts

### Self Protocol — ZK Identity Verification
- **Verifier** (`lib/self/verifier.ts`): `SelfBackendVerifier` with ZK proof validation for age, nationality, OFAC compliance
- **Privacy-preserving**: Only disclosed attributes are revealed; full identity stays private

### Filecoin/Storacha — Decentralized Storage
- **Upload** (`lib/storage/storacha.ts`): Content-addressed storage via `@storacha/client`
- **Retrieve** via IPFS gateway with CID validation

### Uniswap V3 — Pre-Escrow Funding
- **Swap** (`lib/swap/uniswap.ts`): Convert user tokens to USDC on Base before escrow deposit
- Uses QuoterV2, SwapRouter, Factory contracts directly on-chain

## Discovery Waterfall

Counterparty discovery follows a priority-ordered waterfall (`lib/discovery/resolver.ts`):

1. **Seeded Directory** → demo/test counterparties from DB

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mds-main/PACT](https://github.com/mds-main/PACT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
