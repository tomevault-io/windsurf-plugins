---
trigger: always_on
description: > Precision-first context file. Every section here exists to eliminate re-discovery work. Read this before touching any file.
---

# CLAUDE.md — P402 Router

> Precision-first context file. Every section here exists to eliminate re-discovery work. Read this before touching any file.

---

## What This Project Is

**P402** is a production AI payment router and agentic orchestration platform. It sits between AI agents and AI providers, handling:
- Intelligent multi-provider routing (cost / quality / speed / balanced)
- On-chain micropayment settlement via the x402 protocol (EIP-3009 gasless USDC on Base)
- Agent-to-Agent (A2A) and Agent-to-Payment (A2P) protocol over JSON-RPC 2.0
- Autonomous AI governance via Gemini 3 intelligence layer
- ERC-8004 trustless agent identity and reputation

---

## Stack

| Layer | Technology | Version | Notes |
|---|---|---|---|
| Framework | Next.js | 15 | `output: "standalone"`, App Router |
| Language | TypeScript | 5.9.3 | `strict`, `noUncheckedIndexedAccess`, `strictNullChecks` |
| Runtime | Node.js | ≥20 | ESM (`"type": "module"`) |
| UI | React | 19.2.3 | No Pages Router |
| Styling | TailwindCSS | 3.4.3 | CSS vars design system, Neo-Brutalist |
| Web3 (frontend) | Viem | 2.42.0 | Requires explicit `account` + `chain` in `writeContract` |
| Web3 (frontend) | Wagmi | 2.5.7 | |
| Web3 (backend) | Ethers | **v6** | NOT v5 — use `ethers.Provider`, `ethers.keccak256`, `ethers.AbiCoder.defaultAbiCoder()` |
| Wallet UI | RainbowKit | 2.0.0 | |
| Database | PostgreSQL | 8.x (pg) | Neon serverless; pooled via `lib/db.ts` |
| Cache | Redis / ioredis | 5.8.2 | Semantic cache + rate limiting |
| Auth | NextAuth | 4.24.11 | |
| AI — Intelligence | @google/generative-ai | 0.24.1 | Gemini 3 Pro (economist) + Flash (sentinel) |
| AI — Routing | 13 providers | — | See `lib/ai-providers/registry.ts` |
| State | TanStack React Query | 5.25.0 | |
| Testing (unit) | Vitest | 4.0.16 | |
| Testing (e2e) | Playwright | 1.58.2 | |
| Linting | ESLint | 9.0.0 | `extends: "next"` — disabled during builds, run in dev |
| SDK (local) | @p402/sdk | file:sdk | Protocol types, mandate helpers |
| Smart contracts | Hardhat | 2.22.0 | `contracts/` excluded from tsconfig |

---

## Commands

```bash
npm run dev             # Next.js dev server
npm run build           # Production build (standalone)
npm start               # Production server
npm run lint            # ESLint
npm run seed            # Seed DB with initial data
npm test                # Vitest (watch)
npm run test:run        # Vitest (once)
npm run test:coverage   # Coverage report
npm run test:a2a        # A2A protocol integration tests
npm run test:routes     # Route integrity and API exports tests
npm run test:build      # Build smoke test
npm run test:all        # Runs all Vitest tests + build smoke test
npm run test:e2e        # Playwright E2E (all)
npm run test:e2e:smoke  # Playwright E2E smoke tests
npm run test:e2e:resilience # Playwright error resilience tests
npm run test:e2e:ui     # Playwright UI mode
```

---

## File Tree

```
p402-router/
│
├── app/                            # Next.js App Router
│   ├── _components/
│   │   └── TokenSelector.tsx
│   ├── admin/page.tsx              # Admin UI
│   ├── api/
│   │   ├── a2a/                    # ★ A2A JSON-RPC endpoint cluster
│   │   │   ├── route.ts            # Core A2A endpoint (POST — JSON-RPC dispatch)
│   │   │   ├── agents/             # Agent discovery: GET /api/a2a/agents
│   │   │   │   ├── route.ts
│   │   │   │   └── [agentId]/route.ts
│   │   │   ├── bazaar/             # Agent marketplace API
│   │   │   │   ├── route.ts
│   │   │   │   └── [listingId]/route.ts
│   │   │   ├── mandates/           # AP2 mandate issuance + use
│   │   │   │   ├── route.ts
│   │   │   │   └── [id]/use/route.ts
│   │   │   ├── orchestrate/route.ts  # Multi-agent orchestration
│   │   │   ├── stream/route.ts     # SSE streaming for A2A tasks
│   │   │   └── tasks/              # Task lifecycle CRUD
│   │   │       ├── route.ts
│   │   │       ├── [id]/route.ts
│   │   │       └── [id]/route.test.ts
│   │   ├── admin/
│   │   │   ├── health/route.ts     # Admin health check
│   │   │   ├── refresh/route.ts
│   │   │   └── stats/route.ts
│   │   ├── auth/[...nextauth]/route.ts  # NextAuth
│   │   ├── health/route.ts         # Public health endpoint
│   │   ├── internal/
│   │   │   ├── cron/               # Background cron jobs
│   │   │   │   ├── bazaar/sync/route.ts
│   │   │   │   ├── cleanup-replay/route.ts
│   │   │   │   ├── erc8004/feedback/route.ts
│   │   │   │   ├── erc8004/reputation/route.ts
│   │   │   │   └── facilitators/health/route.ts
│   │   │   └── poll/               # Polling endpoints
│   │   ├── openapi.json/route.ts   # OpenAPI spec
│   │   ├── v1/                     # REST API v1
│   │   │   ├── access-request/     # Facilitator access requests
│   │   │   ├── admin/              # security, quarantine
│   │   │   ├── analytics/          # spend, alerts, decisions, conversion
│   │   │   ├── bazaar/             # Agent marketplace, sync, import-route
│   │   │   ├── billing/            # webhook/route.ts (Stripe), upgrade-math/route.ts
│   │   │   ├── cron/               # poll-facilitators
│   │   │   ├── discovery/          # resources
│   │   │   ├── erc8004/            # feedback, reputation, validate

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Z333Q) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
