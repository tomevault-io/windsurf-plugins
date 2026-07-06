---
trigger: always_on
description: > First action of every session: **read this entire file.** Everything you build obeys it.
---

# SLUICE — Project Rules & Constants (read this fully every session)

> First action of every session: **read this entire file.** Everything you build obeys it.
> Working tree is `/root/Sluice` (this is NOT Tessera/Xyndicate/Agora/Apogee/Archon/Auralis —
> never touch those projects). Repo: `Franlinozz/Sluice`. Vercel project: `sluice`
> (`prj_P12m6QjGi3gFtPSCfXbDTiGJRbi9`, team `franlinozzs-projects`).

## What we're building
Sluice is a settlement layer for the agent-paid web. Any unit of value — a read, a
second, a citation, a listen, an API call — is metered and settled on Arc in USDC.
Creators get paid per use; agents pay per use and DECIDE for themselves.
Hero use case: AI research agents pay creators PER CITATION (the open, creator-owned
alternative to Cloudflare Pay-Per-Crawl / RSL, which declare terms but don't settle).

## Non-negotiable rules
1. NEVER fake data, balances, figures, transactions, or "working" integrations. A clearly
   labeled "beta"/"coming soon"/"roadmap" state ALWAYS beats a fake. Every number shown to a
   user must trace to a real on-chain (Arc testnet) event or a real DB record.
2. NO DEAD CONTROLS. Every button/link/toggle either works or is visibly disabled with a
   stated reason (tooltip/label). The product owner is not a deep dev and cannot spot a
   silent no-op — so there must be none.
3. Network-agnostic settlement. All chain/settlement access goes through ONE interface
   (packages/chain). Arc Testnet is primary; swapping the RPC provider (or, last resort, another
   network) must be a config change, never a code rewrite. Do NOT scatter RPC URLs/chain IDs.
4. Meter decoupled from settlement backend. The accrual engine must not care whether the
   final settle is Gateway-batched or a direct x402 call. If Gateway-on-Arc misbehaves, we
   can fall back to direct x402 settlement without touching accrual logic.
5. USDC DECIMALS DISCIPLINE (critical):
   - ERC-20 USDC used for PAYMENTS = 6 decimals. All payment amounts, EIP-3009 values, and
     ledger math use 6-decimal integer base units. Use viem parseUnits(amount, 6).
   - Arc's NATIVE GAS token is USDC displayed with 18 decimals. Never mix gas display with
     payment amounts. Keep a single money helper (packages/money) and route all amounts through it.
6. EIP-3009 is signed against the GATEWAY WALLET CONTRACT, not the USDC token contract.
   The x402 `scheme` field is `"exact"`; the string `"GatewayWalletBatched"` is the EIP-712
   DOMAIN NAME placed in `extra.name`, with `extra.verifyingContract` = the Gateway Wallet.
   (This is the #1 thing teams get wrong — see "x402 payment requirements shape" below.)
7. One-time deposit required: a wallet must deposit USDC into the Gateway Wallet contract
   BEFORE it can make nanopayments. Build and surface this step explicitly.
8. Settlement LAG is real: seller earnings appear only AFTER Circle Gateway settles the batch
   on-chain (can take minutes). UI must show honest states: authorized → batching → settled.
   Never show settled balance the instant a payment is authorized.
9. Money in code is always integer base units (bigint). Format for display only at the edge.
   JSON cannot carry bigint — serialize bigints as strings, parse back at boundaries.
10. Verify before assuming. When unsure of an API shape, address, or version, READ the
    canonical source (links below) or the arc-nanopayments repo — do not invent signatures.
11. Commit after every green Definition of Done, with a clear message. Keep PR-sized diffs.
12. Secrets: server-only keys live in env, never in client bundles. In Next.js, only
    NEXT_PUBLIC_* is exposed to the browser. Private keys/API keys must NEVER be NEXT_PUBLIC_.

## Verified constants (Arc Testnet — CONFIRMED against circlefin/arc-nanopayments, June 2026)
- Chain ID: 5042002 · CAIP-2 network id: `eip155:5042002`
- Primary RPC: https://rpc.testnet.arc.network
- Backup RPCs (same chain, ordered fallback; keep Arc primary, quiet):
  Alchemy https://arc-testnet.g.alchemy.com/v2/<key> · dRPC https://arc-testnet.drpc.org
- Explorer: https://testnet.arcscan.app   (link every tx/receipt here)
- Native gas token: USDC (18-decimal display on Arc). Faucet: ~1 USDC/day — budget it.
- Finality: sub-second (Malachite consensus). EVM-compatible.
- **USDC token contract (ERC-20, 6 decimals, for PAYMENTS):** `0x3600000000000000000000000000000000000000`
- **Gateway Wallet contract (verifyingContract for EIP-3009): `0x0077777d7EBA4688BDeF3E311b846F25870A19B9`**
  ✅ CONFIRMED in arc-nanopayments `lib/x402.ts`. Still read from env in code; do not scatter.
- Circle "domain" id for Arc Testnet (Gateway/CCTP): **26**
- Gateway balances API (testnet): https://gateway-api-testnet.circle.com/v1/balances
  POST `{ token: "USDC", sources: [{ domain: 26, depositor: <addr> }] }`

## x402 payment requirements shape (verified from arc-nanopayments lib/x402.ts)
```
requirements = {
  scheme: "exact",
  network: "eip155:5042002",
  asset: "0x3600000000000000000000000000000000000000", // USDC token, 6dp
  amount: "<atomic base units, 6dp, as string>",        // e.g. $0.001 -> "1000"
  payTo: sellerAddress,
  maxTimeoutSeconds: 345600,
  extra: { name: "GatewayWalletBatched", version: "1",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Franlinozz/Sluice](https://github.com/Franlinozz/Sluice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
