---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run Commands

```bash
# Facilitator (Express + SQLite) — handles all chain interaction
bun run dev                    # Dev mode with watch (port 3000)
bun run start                  # Production start

# Web (Next.js 15) — pure frontend, calls facilitator API
cd web && bun run dev          # Dev mode (port 3001)
cd web && bun run build        # Production build
cd web && bun run start        # Production start

# Local development (two terminals):
# Terminal 1: bun run dev
# Terminal 2: cd web && NEXT_PUBLIC_FACILITATOR_URL=http://localhost:3000 bun run dev

# Contracts (Foundry)
bun run build:contracts        # forge build (from contracts/)
bun run test:contracts         # forge test (from contracts/)
cd contracts && forge test --match-test test_specificName  # Run single test
cd contracts && forge test -vvvv  # Verbose output with traces

# ABI sync (run after contract changes)
bun run build:contracts && bun run sync-abi

# Docker (facilitator only)
docker build -t xenga-facilitator .
docker run -p 8080:8080 --env-file .env xenga-facilitator
```

## Architecture

On-chain escrow and reputation system on Base Sepolia using USDC (ERC-3009 gasless transfers). The Xenga protocol provides the HTTP integration layer. Contracts use **UUPS upgradeable proxy pattern** (ERC1967Proxy + `initialize()`) for safe on-chain upgrades.

**Split deployment:**
- **`web/`** — Next.js 15 frontend deployed on **Vercel**. Pure client-side: pages, wallet management, EIP-712 signing, Protocol Inspector. Calls the facilitator API via `NEXT_PUBLIC_FACILITATOR_URL`.
- **`src/server/`** — Express facilitator deployed on **Fly.io**. Handles all chain interaction: settlement, event listening, reputation, order management, SQLite DB. Runs with `PRIVATE_KEY` for gas.

```
Vercel (web/)                    Fly.io (src/server/)
┌──────────────────┐             ┌──────────────────────────┐
│ Next.js Frontend │   fetch     │ Express Facilitator      │
│ Pages + Signing  │────────────>│ REST API + Chain + SQLite│
└──────────────────┘   CORS      └──────────────────────────┘
        │                                   │
        │ signTypedData                     │ PRIVATE_KEY (gas)
        ▼                                   ▼
   User's Browser                  EscrowVault (Base Sepolia)
```

**Other layers:**
- **`contracts/`** — Foundry project: EscrowVault (escrow state machine + stats + UUPS upgradeable), SessionEscrow (session micropayments + UUPS upgradeable), MockUSDC (test token). Deployed behind ERC1967Proxy — `initialize()` replaces constructors. Storage gap (`uint256[48] private __gap;`) reserves slots for future upgrades.
- **`src/client/`** — Client SDK: EIP-712 signing, reputation lookup (`getReputation()`), and xenga payment flow (`escrowFetch` with optional `onSellerReputation` callback)

**Shared code** (`src/shared/`): types, constants, EIP-712 domain/types, and auto-generated ABIs (`abi.ts` — never edit manually, use `sync-abi`). The web app imports `@shared/` via webpack alias for types and EIP-712 signing functions (client-safe, no server deps).

### Escrow Lifecycle (On-Chain)

```
None → Active → DeliveryConfirmed → Completed      (buyer releases)
         │             │              AutoReleased   (timeout, facilitator poller triggers)
         │             └────────────→ Disputed ──→ Resolved (arbiter splits %)
         └───────────────────────────→ Refunded   (seller voluntary / arbiter)
```

- **Active**: escrow created, USDC locked, `contentHash` stored on-chain (keccak256 of order terms/metadata). Buyer can release anytime, seller/facilitator can confirm delivery or refund.
- **DeliveryConfirmed**: seller/facilitator confirmed delivery, dispute window starts. Buyer can release or dispute.
- **AutoRelease timing**: From Active state, requires `releaseWindow + disputeWindow`. From DeliveryConfirmed, requires `releaseWindow` from creation AND `disputeWindow` from delivery confirmation.
- **Dispute timing**: From DeliveryConfirmed, within `disputeWindow` of confirmation. From Active, between `releaseWindow - disputeWindow` and `releaseWindow + disputeWindow` from creation.
- **disputeWindow**: Owner-settable global default (default: 3 days, bounds: 1 hour–30 days). Set via `EscrowVault.setDisputeWindow()`. Stored per-escrow at creation — existing escrows keep their original value.
- **Resolved**: arbiter splits funds by buyer percentage (0-100). Fee goes to feeRecipient, split applies to `amount - fee`.
- **Refunded**: buyer gets full deposit back including fee — facilitator absorbs cost.

### Reputation System

On-chain credit scoring for agents/wallets, computed from escrow transaction history. The facilitator tracks both `sellerStats` and `buyerStats` on-chain (in `EscrowVault.sol`), and the server computes weighted reputation scores from this data.

**Architecture:**
- **On-chain**: `buyerStats[address]` and `sellerStats[address]` mappings in EscrowVault track totalEscrows, completedCount, disputedCount, refundedCount, resolvedCount, and amounts. Raw data is permissionless — anyone can read and compute their own scores.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uhyunpark/xenga](https://github.com/uhyunpark/xenga) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
