---
trigger: always_on
description: Guidance for AI coding agents (and humans) working in this repo. PipRail is an **open, backendless, no-fee** SDK for x402 "402 Payment Required" crypto payments across 37 chains, plus a static marketing site.
---

# AGENTS.md

Guidance for AI coding agents (and humans) working in this repo. PipRail is an **open, backendless, no-fee** SDK for x402 "402 Payment Required" crypto payments across 37 chains, plus a static marketing site.

## Commands

```bash
npm install              # install workspaces (sdk, site)
npm run build:sdk        # build the SDK
npm run test:sdk         # SDK test suite (Vitest) — one number over every suite
npm run sweep            # the SAME suite in 12 named sections, one line each
npm run sweep -- swaps   # just one section (also: --list, --files, --bail)
npm run smoke            # adversarial (L2) + live read-only (L3) — see TESTING.md
npm run smoke -- --money # adds L4: real mainnet payments. Spends money.
npm run typecheck        # typecheck the SDK
npm run dev              # run the site locally → http://localhost:4321
npm run build            # build the static site
```

Per-example (each is standalone): `cd examples/<name> && npm install && npm start`.

## Project structure

```
sdk/        @piprail/sdk — the product (the only npm-published package)
  src/index.ts          public API surface
  src/server.ts         requirePayment / createPaymentGate (accept side)
  src/client.ts         PipRailClient (pay side)
  src/agent.ts          paymentTools (agent side)
  src/x402.ts           wire protocol (challenges, receipts) — chain-agnostic
  src/policy.ts ledger.ts errors.ts util/
  src/drivers/          one folder per chain family: evm solana ton stellar xrpl tron near sui aptos algorand
    types.ts            the PaymentDriver contract (the only thing the protocol layer sees)
  test/                 Vitest — the contract
  README.md ERRORS.md STANDARDS.md
examples/   teaching code (standalone): express/ next-app-router/ agent/ mcp/ + README.md + CONCEPTS.md
integrations/ first-party framework integrations (one folder per framework, standalone + publishable): openclaw/piprail/ (ClawHub skill @piprail) + hermes/piprail/ (Hermes MCP catalog manifest + Skills Hub skill) + elizaos/piprail/ (@piprail/elizaos-plugin) + n8n/piprail/ (@piprail/n8n-nodes-piprail) + mastra/piprail/ (MCPClient example); MCP-based ones ship verify.mjs; + README.md + TESTING.md
site/       piprail.com — Astro 5 + Tailwind v4 (deploys to Netlify)
```

## API ground truth (get these right)

- **Accept, Express/Connect only:** `requirePayment({ chain, token, amount, payTo })` → middleware. `token` is **required**; there is no default.
- **Accept, every other framework:** `createPaymentGate({ chain, token, amount, payTo })` → `gate.verify(headerValue)` returns `{ kind: 'paid' | 'challenge' | 'invalid', … }`. Use `toInvalidBody(result)` for the invalid 402 body.
- **Receipts:** `onPaid(receipt)` fires on a settled payment with an enriched **`PaidReceipt`** (`X402Receipt` + `decimals`/`symbol`/`amountFormatted`/`idempotencyKey`). It's **sync or async and fully isolated** (a throw OR a rejected promise → `onPaidError`, never crashes); fire-and-forget unless `awaitOnPaid: true`. Delivery is **at-least-once** — dedupe on `idempotencyKey`. `deliverReceipt(receipt, { url, secret })` is a never-throws, signed+retried POST to **your** webhook (PipRail hosts nothing).
- **Multi-chain:** `{ accept: [{ chain, token, amount, payTo? }, …] }` instead of the single-chain fields.
- **Pay:** `new PipRailClient({ chain, wallet, policy? })` → `client.fetch(url)` auto-pays a 402; `quote(url)`, `estimateCost(url)`, **`planPayment(url)`** (affordability + recipient-readiness preflight → `PaymentPlan`; `canAfford(url)`; `fetch(url, { autoRoute: true })` pays the cheapest settleable rail), `spent()` / `budget()` / `policy()`. Module-level `planAcross(clients, url)` plans across chains.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [piprail/piprail](https://github.com/piprail/piprail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
