---
trigger: always_on
description: **ETHGlobal New York 2026** · anti-scalp NFT tickets on Hedera
---

# Hedera World Cup Ticket — Hackathon Onboarding

**ETHGlobal New York 2026** · anti-scalp NFT tickets on Hedera

---

## 1. What this is

Each World Cup ticket is an NFT on Hedera. When the organizer creates a collection they bake in a **10% royalty fee**: every time the NFT moves in an atomic swap (NFT transferred in exchange for HBAR in the same transaction), the organizer gets 10% of the sale price automatically and without any app-level code. Authenticity is cryptographically guaranteed — no counterfeits.

**Mint-on-buy:** tickets are not pre-minted. The organizer sets `maxSupply` and **face value** only. When a purchaser buys, the backend mints one serial and transfers it in the same flow.

**World ID gates:**
- **Onboarding / login** — one nullifier → one Hedera account (Sybil protection at signup).
- **Secondary purchase** — buyer must verify World ID; nullifier stored so one human = one secondary purchase (configurable cap).

**What this is NOT:** there is no hard on-chain price cap on resales. The royalty and the World ID cap together create strong disincentives for scalping, but a determined buyer can still overpay once. That framing matters for the pitch.

---

## 2. Prize targets

| Prize | Track | Why we qualify |
|---|---|---|
| Hedera Tokenization | $3,000 — anchor | Custom fees / royalties, NFT mint, on-chain transfer |
| World ID Track B | $2,500 | Onboarding + login + resale require World ID; nullifier enforces one wallet + secondary cap |
| ENS pool | Small bolt-on | ENS subdomain as a human-readable ticket address |
| Hedera No-Solidity | $3,000 — fallback | HCS used as an audit log (second native Hedera service, zero Solidity) |

---

## 3. Tech stack

- **Runtime:** Node.js, ES modules (`"type": "module"` in package.json)
- **App:** Next.js 16 (App Router) — single server for UI + API routes
- **Hedera SDK:** `@hashgraph/sdk` v2.81+
- **Network:** Hedera testnet (custodial accounts, operator pays all fees)
- **Payment:** HBAR for the core sale; USDC planned later
- **Proof of human:** `@worldcoin/idkit` v4 — IDKit widget + backend verify via `/api/v4/verify/{rp_id}`
- **Database:** SQLite (`better-sqlite3`) — users, tokens, tickets, ownership in `data/users.db`

> **SDK name trap:** The Hedera Playground and older snippets use `@hiero-ledger/sdk`. This repo uses `@hashgraph/sdk`. The API is identical; only the import changes. Don't mix them.

---

## 4. Architecture & conventions

```
app/
  page.jsx                              ← Marketplace (mint-on-buy at face value)
  login/page.jsx                        ← Returning user: World ID → existing account
  onboard/page.jsx                      ← New user: World ID → create account + role
  wallet/page.jsx                       ← My tickets, resale UI
  organizer/page.jsx                    ← Create collections (maxSupply + faceValueHbar)
  components/Nav.jsx                    ← Global nav, log out
  lib/storage.js                        ← localStorage session (ticket_account_id)
  api/
    login/route.js                      ← verify proof → lookup existing user
    verify-and-onboard/route.js         ← verify proof → createUserAccount(60)
    world-id/sign/route.js              ← RP signature for IDKit v4
    marketplace/route.js                ← collections with faceValueHbar, remaining
    tokens/route.js                     ← POST create collection (organizer)
    tokens/[tokenId]/buy/route.js       ← mint-on-buy primary purchase
    tickets/[tokenId]/[serial]/resell/  ← atomic resale + World ID on buyer
    wallet/[accountId]/route.js         ← tickets + ownership history

src/
  client.js                             ← shared Hedera client factory
  state.js                              ← loadState() / saveState() for state.json
  lib/auth.js                           ← requireUser(), requireRole()
  db/
    schema.sql                          ← users, tokens, tickets, ownership
    users.js, tokens.js, tickets.js, db.js
  world/
    verifyProof.js                      ← verifyWorldIdProof(), extractNullifier()
  hedera/
    createToken.js                      ← createTicketToken() — NFT collection + royalty
    createAccount.js                    ← createUserAccount(initialHbar) — custodial wallet
    mintTicket.js                       ← mintTickets(tokenId, supplyKey, pointers)
    primaryPurchase.js                  ← mint → primarySale → DB (sold_primary)
    transferTicket.js                   ← primarySale(), atomicResale()
    compliance.js                       ← freezeHolder/unfreezeHolder/pauseToken/unpauseToken

scripts/
  01-check-balance.js                   ← prove .env + network; print operator HBAR
  02-create-token.js                    ← [maxSupply] [faceValueHbar] [name] [symbol]; requires operator seeded in DB (run app once first)
  03-create-account.js                  ← dev bypass: create buyer without World ID; writes buyer → state.json
  04-mint.js                            ← DEPRECATED (superseded by mint-on-buy in 05)
  05-primary-sale.js                    ← CLI mint-on-buy; requires tokenId + buyer in state.json
  06-resale.js                          ← CLI atomic resale; args: [serial] [priceHbar] [sellerAccountId] [buyerAccountId]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vedantMahangade/fake-repo](https://github.com/vedantMahangade/fake-repo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
