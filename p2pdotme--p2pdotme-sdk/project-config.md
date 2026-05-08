---
trigger: always_on
description: > **Intentionally committed.** This file provides AI tooling context for contributors. It contains no secrets and is excluded from the npm tarball via `files: ["dist"]` in `package.json`.
---

# CLAUDE.md

> **Intentionally committed.** This file provides AI tooling context for contributors. It contains no secrets and is excluded from the npm tarball via `files: ["dist"]` in `package.json`.

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

`@p2pdotme/sdk` — a multi-module TypeScript SDK for P2P.me. Published as a single package with subpath exports:

- `@p2pdotme/sdk/orders` — full order surface: reads (`getOrder`, `getOrders`, `getFeeConfig`) + writes via `prepare`/`execute` pairs (`placeOrder`, `cancelOrder`, `setSellOrderUpi`, `raiseDispute`, `approveUsdc`, `paidBuyOrder`) + event subscriptions (`watchEvents`). Circle-selection routing lives inside as an internal implementation detail.
- `@p2pdotme/sdk/prices` — currency price config reads: `getPriceConfig`, `getReputationPerUsdcLimit`.
- `@p2pdotme/sdk/profile` — user-scoped reads: USDC balance, USDC allowance, tx limits, combined fiat balances.
- `@p2pdotme/sdk/react` — unified React provider (`SdkProvider`) + hooks (`useOrders`, `usePrices`, `useProfile`, `useZkkyc`, `useFraudEngine`, `useSdk`).
- `@p2pdotme/sdk/qr-parsers` — QR code parsers for payment networks (UPI, PIX, QRIS, MercadoPago, Pago Móvil).
- `@p2pdotme/sdk/fraud-engine` — fraud detection, device fingerprinting (FingerprintJS), SEON session signals, encrypted activity logging.
- `@p2pdotme/sdk/zkkyc` — ZK KYC: tx calldata preparation for Reclaim social verify, Aadhaar, ZK Passport; plus UX flow orchestrators.
- `@p2pdotme/sdk/country` — country/currency metadata, payment field configs, per-currency validators.

Framework-agnostic core. Wallet-agnostic — consumers bring their own viem `PublicClient` (reads) and optionally a `WalletClient` (writes).

## Prerequisites

- Node.js 22.4.1+ (see `.nvmrc`)
- Bun 1.3.1 (pinned via `engines` and `packageManager` in `package.json`)

## Commands

```bash
bun run build          # tsup → ESM + CJS + DTS into dist/
bun run dev            # tsup --watch
bun run typecheck      # tsc --noEmit (covers src/, test/, example/)
bun run lint           # biome check src/
bun run lint:fix       # biome check --write src/
bun run format         # biome format --write src/
bun run test           # vitest run
bun run changeset      # create a changeset
bun run release        # build + publish
```

## Architecture

```
src/
├── types/                  # PublicClientLike shared across modules
├── constants/              # CURRENCY, ORDER_TYPE, ORDER_STATUS, DISPUTE_STATUS — internal only
├── lib/                    # Shared low-level utilities — internal only
│   ├── encoding.ts         # bytesToBase64, hexToBytes
│   ├── logger.ts           # Logger type + noopLogger
│   ├── sleep.ts
│   └── subgraph.ts         # querySubgraph() + SubgraphError
├── contracts/              # Centralized contract interactions (no duplicate ABIs)
│   ├── abis/               # ABI fragments (order-flow, order-processor, p2p-config, reputation-manager)
│   ├── order-flow/         # checkCircleEligibility
│   ├── order-processor/    # readOrderMulticall, readFeeConfigMulticall
│   ├── p2p-config/         # getPriceConfig, getReputationPerUsdcLimit
│   ├── tx-limits/          # getTxLimits
│   ├── usdc/               # getUsdcBalance
│   └── reputation-manager/
├── validation/             # SdkError base + shared Zod schemas (ZodAddressSchema, ZodCurrencySchema)
├── orders/                 # @p2pdotme/sdk/orders
│   ├── client.ts           # createOrders() — reads + write actions on one flat surface
│   ├── errors.ts           # unified OrdersError with read + write codes
│   ├── types.ts            # Order, OrderStatus, OrdersConfig, PreparedTx, TxResult, ExecuteBase
│   ├── validation.ts       # Zod schemas for every read + write param
│   ├── normalize.ts        # contract + subgraph → normalized Order
│   ├── tx.ts               # submitPreparedTx helper
│   ├── internal/
│   │   └── routing/        # Epsilon-greedy circle selection (not exported publicly)
│   ├── actions/            # prepare/execute pairs per action
│   │   ├── place-order.ts
│   │   ├── cancel-order.ts
│   │   ├── set-sell-order-upi.ts
│   │   ├── raise-dispute.ts
│   │   └── approve-usdc.ts
│   ├── relay-identity/     # Pure createRelayIdentity, in-memory + localStorage stores, resolver
│   ├── crypto/             # ECIES + encryptPaymentAddress/decryptPaymentAddress
│   └── subgraph/           # OrdersForUser query
├── prices/                 # @p2pdotme/sdk/prices — currency price config reads
├── profile/                # @p2pdotme/sdk/profile — user-scoped balance + limits
├── qr-parsers/             # UPI, PIX, QRIS, MercadoPago, Pago Móvil
├── payload/  ← REMOVED — merged into orders (prepare/execute)
├── order-routing/ ← REMOVED — moved to orders/internal/routing/
├── order-actions/ ← REMOVED — merged into orders
├── fraud-engine/
├── zkkyc/                  # createZkkyc + createReclaimFlow + createZkPassportFlow
├── country/                # COUNTRY_OPTIONS, PAYMENT_ID_FIELDS, per-currency validators
└── react/                  # SdkProvider + hooks
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [p2pdotme/p2pdotme-sdk](https://github.com/p2pdotme/p2pdotme-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
