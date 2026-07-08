---
trigger: always_on
description: **The card-privacy backend is now Arcium MPC, NOT Inco TEE + MagicBlock VRF.** The
---

# HiddenHand - Privacy Poker on Solana

## ⚡ CURRENT STATE (2026) — Arcium MPC (READ THIS FIRST)

**The card-privacy backend is now Arcium MPC, NOT Inco TEE + MagicBlock VRF.** The
hackathon-era sections further down describe the retired Inco/VRF design and are kept
only as historical context. Where they conflict with this banner, this banner wins.

**Why the switch:** the old design had a SEV-HIGH flaw — the deck was reconstructable
from the public VRF callback randomness, so "encryption" was theater against a chain
observer. Arcium shuffles inside MPC; randomness never touches the chain; the deck lives
on-chain only as an opaque MXE ciphertext.

**Architecture (the card lifecycle = 6 Arcium MPC circuits in `encrypted-ixs/src/lib.rs`):**
- `shuffle` → shuffles the 52-card deck in-MPC, seals it to the MXE, stored on-chain in
  `DeckState.deck` (`[[u8;32];2]`) + `deck_nonce`. Re-fed unchanged into every later circuit.
- `deal_to_seat` (once per seated player) → seals that seat's 2 hole cards to the player's
  own x25519 key. Emitted via the `HoleDealt` event; only that player can decrypt (client-side
  RescueCipher). Deck positions `2i, 2i+1` for seat `i`; board at 18–22.
- `reveal_flop` / `reveal_turn` / `reveal_river` → re-feed the deck, `.reveal()` the board
  publicly; callback writes `HandState.community_cards` and advances the phase.
- `showdown_reveal` → batched; reveals non-folded hole cards (mask from on-chain fold state)
  into each seat's `revealed_card_1/2`; the existing `showdown` eval then runs on public cards.

Retired: `request_shuffle`/`callback_shuffle` (VRF), `encrypt_hole_cards`, `grant_card_allowance`,
`grant_own_allowance`, `grant_community_allowances`, `reveal_cards`/`reveal_community` (Inco/Ed25519),
`inco_cpi.rs`, `ephemeral-vrf-sdk`. Betting / pot / side-pots / rake / hand-eval are unchanged
public Solana logic (still 48 passing unit tests).

**Version stack:** `arcis`/`arcium-anchor`/`-client`/`-macros` `=0.11.1`, `anchor-lang 1.0.x` /
Solana 3.x, `arcium` CLI 0.11.2, `@arcium-hq/client 0.11.2` (frontend), devnet cluster offset **456**.

**Deployment status (Phase 3 complete):** program `9chPz3vJDeU7gr4zBtDreJUpVLKbqwrKoQBQQjT1SF5X`
deployed to **devnet** with MXE + all 6 comp-defs initialized. A full hand has run end-to-end
through the live MPC network (shuffle → deal → bet → reveals → showdown), with dealt cards
matching the showdown-revealed cards and the pot conserved. Circuits are hosted (OffChain source)
at `github.com/criptocbas/hiddenhand-arcium-circuit` — **if you edit a circuit you MUST
`arcium build` → re-push the exact `.arcis` to that repo → redeploy, or computations abort.**

**Frontend:** `app/lib/arcium.ts` (replaces `lib/inco.ts`) — x25519 key derivation, MXE pubkey,
RescueCipher decrypt, Arcium queue-account set, event scanning. `usePokerGame.ts` drives the 6
MPC steps (public API preserved). Builds under Turbopack via `next.config.ts` aliases
(anchor-core-shim + crypto/fs/child_process polyfills — see the file). Reproducible integration
test: `app/scripts/devnet-full-hand.cjs`. RPC must be reliable (Helius) via `NEXT_PUBLIC_SOLANA_RPC`;
public devnet drops Arcium txs.

---

## Conversation Context (IMPORTANT - READ FIRST)

> ⚠️ Historical (hackathon-era Inco/VRF design) — superseded by the Arcium banner above.

This project was started for the **Solana Privacy Hack** hackathon (Jan 12-30, 2025). The user and Claude collaboratively designed and built the initial program structure.

### Key Decisions Made
1. **Project Choice**: We evaluated 5 privacy project ideas and chose **Privacy Poker** because:
   - User's escrow3 experience translates well (state machines, multi-party)
   - Clear scope (Texas Hold'em rules are standardized)
   - Exciting demo potential
   - Targets MagicBlock ($5K) + Open Track ($18K) = $23K bounty potential

2. **Privacy Approach**: Hybrid **MagicBlock VRF + Inco TEE** for ultimate privacy:
   - **MagicBlock VRF**: Provably fair card shuffling with verifiable randomness
   - **Inco TEE**: confidential computing (TEE) for card privacy
     - Cards encrypted as u128 handles on-chain
     - Only card owner can decrypt (via allowances)
     - Ed25519 signature verification for reveals
     - **Cryptographic guarantee**: Cards are ALWAYS encrypted, even during computation

3. **Manual Inco CPI**: Built custom CPI module (`src/inco_cpi.rs`) for Inco integration to avoid SDK version conflicts.

### What's Built
- Full poker game state machine (7 phases)
- Table creation with configurable blinds
- Player join/leave with USDC buy-in (SPL token support)
- Betting logic (Fold/Check/Call/Raise/AllIn)
- Pot management and action rotation
- Card encoding (0-51) with Inco TEE encryption
- **Hand evaluation algorithm** (best 5 from 7 cards)
- **Showdown with pot distribution** (handles split pots)
- **48 passing unit tests**
- **MagicBlock VRF Integration** - Provably fair card shuffling
- **Inco TEE Encryption** - Hole cards encrypted as u128 handles
- **Ed25519 Signature Verification** - Secure card reveals at showdown
- **Complete Next.js Frontend** - Playable poker UI with wallet integration
- **Client-side Decryption** - Players decrypt their own cards via Inco SDK

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HiddenHandPoker/HiddenHand](https://github.com/HiddenHandPoker/HiddenHand) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
