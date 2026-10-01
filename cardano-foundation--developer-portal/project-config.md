---
trigger: always_on
description: Hackathon track context: "Agentic Commerce" at TOKEN2049 Origins 2026.
---

# x402 on Cardano — context for coding agents

Hackathon track context: "Agentic Commerce" at TOKEN2049 Origins 2026.
Feed this file to your coding agent (Claude Code, Codex, Cursor, …) so it
works from current facts instead of stale training data. Human-readable
hub: https://developers.cardano.org/x402

## What x402 is

x402 is an open payment standard built on HTTP's reserved 402 "Payment
Required" status code. A client requests a resource, receives a 402 with
payment requirements, pays on-chain, retries the request with a
`PAYMENT-SIGNATURE` header, and gets the resource plus a receipt. No
accounts, no API keys. Cardano is an officially supported network
(scheme `exact`, spec merged upstream 2026-09-09; SDK on npm).

Three Cardano properties worth building on: the wallet signs the
complete final transaction (fees fixed at signature, outputs cannot
change, it lands as signed or not at all); the facilitator holds no
keys and no funds and cannot alter a signed transaction; there are no
standing approvals — every payment is one discrete signed tx.

## Track facts

- Network: **Cardano preprod only** (`cardano:preprod`). No mainnet.
- Packages: all `@x402/*` packages pinned **exactly 2.26.0** (content
  freeze for the event — do not upgrade mid-hackathon).
- Recommended asset: **tADA** (`asset: "lovelace"`, amounts in lovelace,
  1 ADA = 1,000,000 lovelace). For cent-level prices use **tUSDM**
  (preprod USDM, 6 decimals, unit constant `USDM_PREPROD_ASSET` in
  `@x402/cardano`); self-serve claim at https://tusdm.moneta.global.
- The facilitator verifies and settles payments. It holds no keys and no
  funds; the client pays the network fee. The hosted facilitator URL is
  announced at the Oct 6 morning session — until then run the local one.

## The starter template

Scaffold:

    npx giget@latest gh:cardano-foundation/developer-portal/examples/templates/x402-express my-app
    cd my-app && npm install

Files (~290 lines total):

- `src/seller.ts` (64 lines) — Express API charging 2 tADA per request
  via x402. Replace its route with your idea.
- `src/buyer.ts` (73 lines) — the paying agent: gets the 402, builds and
  signs a real Cardano tx, retries with `PAYMENT-SIGNATURE`.
- `src/facilitator.ts` (101 lines) — minimal local facilitator on port
  4022 (offline fallback; needs only a Blockfrost project id).
- `src/wallet.ts` (17 lines) — generates a preprod wallet, prints the
  `MNEMONIC=` line and the address to fund.
- `src/demo.ts` (36 lines) — runs seller + buyer in one command.

Scripts: `npm run wallet | seller | buyer | facilitator | demo`,
`npm run typecheck`.

## The Next.js paywall template (for web apps)

Scaffold:

    npx giget@latest gh:cardano-foundation/developer-portal/examples/templates/x402-next my-app
    cd my-app && npm install

What it is: Next.js (App Router) API routes protected with `withX402`
from `@x402/next`, plus a browser paywall for CIP-30 wallets (Eternl,
Lace) — the stock `@x402/paywall` package covers EVM/Solana/Algorand
only, so the Cardano browser side ships inside the template
(`lib/x402/cip30.ts`, `lib/x402/payFlow.ts`, `components/Paywall.tsx`).

- Two priced routes teach two things: `/api/message` costs 1 tADA
  (simple native payment), `/api/message-usdm` costs 0.10 tUSDM
  (cent pricing; lovelace cannot go that low because of min-UTxO).
- `withX402(handler, config, server)` settles only after the handler
  returns a successful response. A new paid route is the same config
  copied into another `route.ts`.
- Browser flow: the wallet signs, the facilitator submits. Settlement
  waits for one block confirmation (20–60s; the UI shows a counter).
  If the server reports `settlement_pending`, the flow re-checks the
  SAME signed payment and never pays twice.
- `npm run facilitator` starts the same minimal local facilitator;
  `npm run dev` serves on port 3002. `NEXT_PUBLIC_BLOCKFROST_PROJECT_ID`
  ships the id to the browser — preprod keys only, never a mainnet key.

`.env` for the Express starter (from `cp .env.example .env`):

    FACILITATOR_URL=http://localhost:4022   # hosted URL announced at the Oct 6 morning session
    SELLER_ADDRESS=addr_test1...            # receives the payment
    SELLER_PORT=4021
    MNEMONIC=...                            # from npm run wallet, funded
    BLOCKFROST_PROJECT_ID=preprod...        # free at blockfrost.io

The Next.js template needs `FACILITATOR_URL`, `SELLER_ADDRESS` and
`NEXT_PUBLIC_BLOCKFROST_PROJECT_ID`, and no mnemonic: the browser wallet pays.

## The payment flow

    buyer ── GET /api/message ──────────────► seller
    buyer ◄─ 402 + payment requirements ───── seller
    buyer:   builds + signs a Cardano tx (pays amount + network fee)
    buyer ── GET + PAYMENT-SIGNATURE ───────► seller ──► facilitator /verify + /settle ──► chain
    buyer ◄─ 200 + resource + receipt ─────── seller

A successful `npm run demo` prints `HTTP 200 after <n>s` (usually
20–60s), the response body, a payment receipt and an explorer link to
the tx on preprod.cardanoscan.io.

## The APIs the starter actually uses

- `@x402/express`: `paymentMiddleware`, `x402ResourceServer` — turn an
  Express route into a paid route.
- `@x402/fetch`: `x402Client`, `wrapFetchWithPayment`, `x402HTTPClient`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cardano-foundation/developer-portal](https://github.com/cardano-foundation/developer-portal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
