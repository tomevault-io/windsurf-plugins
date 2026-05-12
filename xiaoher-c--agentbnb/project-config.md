---
trigger: always_on
description: AgentBnB is a P2P agent capability sharing protocol. Agent owners publish what their agents can do (Capability Cards) and request capabilities from others, with a lightweight credit-based exchange system. Think Airbnb for AI agent pipelines.
---

# CLAUDE.md — AgentBnB

## Project Overview

AgentBnB is a P2P agent capability sharing protocol. Agent owners publish what their agents can do (Capability Cards) and request capabilities from others, with a lightweight credit-based exchange system. Think Airbnb for AI agent pipelines.

**Core Insight: The user of AgentBnB is not the human. The user is the agent.** (See [AGENT-NATIVE-PROTOCOL.md](AGENT-NATIVE-PROTOCOL.md) for the full design philosophy.)

**Founder**: Cheng Wen Chen
**Domain**: agentbnb.dev
**IP**: © 2026 Cheng Wen Chen, MIT License
**Primary Language**: TypeScript (Node.js)
**Package Manager**: pnpm

## Current State

- **Version**: 1.0.0 (V1.0 conceptual restart — see [docs/V1.0-RESET.md](docs/V1.0-RESET.md))
- **Internal lineage** (preserved for context): v1.1 → v2.x → v3.0 (SkillExecutor, Conductor, Signed Escrow) → v3.1 (WebSocket Relay) → v4.0 (Agent Economy Platform) → v5.0 (Genesis Flywheel) → v5.1 (OpenClaw Hardening) → v6.0 (Team Formation Protocol) → v7.0 (Agent Economy Infrastructure) → v8.x (V8 Identity Convergence) → v9.x (Agent Identity Protocol). V1.0 reframes this as one coherent product.
- **V1.0 capabilities**: Three-layer identity stack (DID + UCAN + Verifiable Credentials) operational
  - DID Envelope (did:key + did:agentbnb, rotation, revocation, EVM bridge) ✅
  - UCAN Token Engine (create/verify/delegate, escrow binding, gateway/relay/conductor integration) ✅
  - Verifiable Credentials (reputation/skill/team VCs, weekly scheduler, selective disclosure) ✅
  - Cross-Platform Federation (DID rotation, VC presentation, EVM bridge) ✅
  - BLS Team Proofs → roadmap (post-V1.0)
- **Tests**: 1,800+

## Tech Stack

- Runtime: Node.js 20+
- Language: TypeScript (strict mode)
- Database: SQLite (via better-sqlite3, WAL mode) for local registry + credits
- Protocol: JSON-RPC over HTTP for agent-to-agent communication
- Testing: Vitest
- Linting: ESLint + Prettier
- Hub: React 18 + Vite + Tailwind CSS (premium dark SaaS theme, served at `/hub`)
- Background Jobs: croner (cron scheduling)
- Events: typed-emitter
- AI: @anthropic-ai/sdk (Claude API for Conductor NLP decomposition)
- MCP: @modelcontextprotocol/sdk (stdio-based MCP server, 6 tools)
- WebSocket: @fastify/websocket (relay system)

## Architecture

```
src/
├── registry/    # Card storage, FTS5 search, health-checker, pricing, credit-routes, openapi (22 files)
├── gateway/     # Agent-to-agent HTTP + batch execution (11 files)
├── credit/      # Ledger, escrow, vouchers, economic system, cross-machine credits (20+ files)
├── runtime/     # Agent lifecycle, ProcessGuard, ServiceCoordinator (7 files)
├── relay/       # WebSocket relay for zero-config P2P networking (6 files)
├── hub-agent/   # Hub-hosted agent management, job queue, relay bridge (13 files)
├── feedback/    # Reputation & feedback scoring (8 files)
├── evolution/   # Agent skill evolution tracking (7 files)
├── auth/        # UCAN tokens, canonical JSON (RFC 8785), resource URI parser (10 files)
├── credentials/ # Verifiable Credentials engine, reputation/skill/team VCs, scheduler (11 files)
├── identity/    # Agent identity, DID (did:key + did:agentbnb), rotation, revocation, EVM bridge, guarantor (15 files)
├── sdk/         # Consumer/Provider SDK for LangChain/CrewAI/AutoGen (7 files)
├── mcp/         # MCP server — tools: discover, request, publish, status, conduct, serve_skill
├── app/         # AgentBnB service entry point
├── onboarding/  # Advanced onboarding (auto-detect from docs, capability templates)
├── autonomy/    # Tier-based autonomy, idle monitor, auto-request (10 files)
├── openclaw/    # OpenClaw integration (SOUL.md sync, heartbeat rules)
├── skills/      # SkillExecutor (5 modes: API, Pipeline, OpenClaw, Command, Conductor)
├── conductor/   # Multi-agent orchestration, team formation, role schema (19 files)
├── utils/       # Shared utilities (interpolation)
├── discovery/   # mDNS peer discovery
├── cli/         # CLI: init, publish, discover, request, serve, quickstart, conduct, mcp-server, did, vc (19 files)
└── types/       # Core TypeScript types + Zod schemas

hub/             # React SPA at /hub (Vite + Tailwind, premium dark theme)
├── pages/       # Discover, Agents, CreateAgent, AgentDashboard, Genesis, CreditPolicy
├── components/  # 40+ components (cards, charts, hero sections, trust badges)
└── hooks/       # useCards, useAuth, useOwnerCards, useRequests

skills/agentbnb/ # OpenClaw installable skill package
```

## Capability Card Schema

Multi-skill cards — one card per agent, multiple independently-priced skills.

Key fields: `id`, `owner`, `name`, `skills[]`, `pricing`, `availability`, `capability_type`, `performance_tier`, `authority_source`, `gateway_url`

Per-skill fields: `capability_types[]`, `requires_capabilities[]`, `visibility` ('public'|'private'), `capacity.max_concurrent`

Full interfaces: `src/types/index.ts` (CapabilityCard, CapabilityCardV2, Skill)

## Agent Identity Protocol

Three-layer identity stack for autonomous agents:

### Layer 1: Cryptographic Identity (DID)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Xiaoher-C/agentbnb](https://github.com/Xiaoher-C/agentbnb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
