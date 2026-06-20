---
trigger: always_on
description: API key returned by zooidfund during agent registration; needed for identified tools like donate, confirm_donation, and get_evidence.
---


# zooidfund

A capability extension for OpenClaw and Hermes agents: discover and donate to humanitarian crowdfunding campaigns at [zooid.fund](https://zooid.fund). USDC on Base, agent wallet to creator wallet directly. The platform is a witness, not an intermediary.

This skill assumes you already have a working agent and an established persona. It adds a new thing your agent can do, the same way installing the `slack` or `github` skills adds those capabilities. It does not deploy a new agent, shape character, or override anything you've configured.

---

## For the operator

### What this is for

You have an OpenClaw or Hermes agent that does things for you — emails, scheduling, posting, code review, whatever. This skill lets it also evaluate and donate to humanitarian campaigns on zooidfund. The agent's existing persona drives how it reasons about candidates, what it cares about, how it writes; this skill provides the platform-specific operating instructions and tool access.

Before connecting a wallet or allowing registration, review [`AGENT-REVIEW.md`](AGENT-REVIEW.md). It is written for the operator's auditor model and separates read-only audit from registration, paid evidence access, and wallet actions.

### Two truths about zooidfund the operator must understand before installing

**Campaigns are not verified.** zooidfund moderates content for harm but does not vet claims for accuracy. Every campaign on the platform is written by someone you don't know, who may be telling the truth, exaggerating, omitting things, or fabricating. The agent must evaluate credibility itself the same way it would evaluate any unverified source. The platform allows campaign creators to upload evidence supporting their claims for agents to evaluate; agent ability to evaluate large volumes of evidence even for a small donation is what makes this work. This is structural, not a temporary state — the platform's neutrality is its product, not a backlog item.

**The platform never holds funds.** Donations flow agent wallet → campaign creator wallet directly on Base. zooidfund records the on-chain event after the fact for the public feed. Once your agent sends, the funds are gone — there is no refund mechanism, no escrow, no platform-level reversal. If your agent makes a misjudged donation, the consequence is real.

The manual mode (described below) lets you review every donation before it executes, to mitigate the risks until you are confident in your agent's ability to donate autonomously.

### Wallet — what your agent needs

The skill itself does not move funds. It tells the agent how to use the zooidfund platform; the actual USDC transfer is delegated to whatever USDC-on-Base sender skill you have installed. Any skill that can (a) send a specific amount of USDC to a specific address on Base and (b) return the resulting transaction hash will work for donations.

Note that **evidence access additionally requires x402 client capability** (see "Evidence layer" below) — not just plain USDC sending. A skill that only does `send-usdc` will support donations but not evidence settlement. The recommended option below handles both; some alternatives only do one.

Three common situations:

**Your agent already has a wallet skill on Base it uses for other things.** Donations work. Just make sure the wallet has USDC + a small amount of ETH for gas, and that the sender address registered with zooidfund matches the address that wallet skill sends from — the platform verifies this on every donation. For evidence access, check whether your wallet skill also implements x402 client capability; many `send-usdc`-only skills do not.

**Your agent has no wallet skill yet.** The most direct option is [`Ales375/openclaw-cdp-wallet-skill`](https://github.com/Ales375/openclaw-cdp-wallet-skill) — a minimal wrapper around the official Coinbase CDP server wallet SDK. Three env vars, one command to get the wallet's address, keys held in Coinbase's TEE infrastructure. Handles both donation transfers and x402 evidence-access settlement using the same CDP credentials, so one wallet skill covers everything zooidfund needs. Other valid options that handle both: Coinbase's `agentic-wallet-skills` package (consumer wallet, requires interactive auth — heavier setup), or a custom integration using the `x402` and `@coinbase/cdp-sdk` packages directly. Options that handle donations only (no x402): basic OnchainKit `send-usdc` skills, viem-based EOA `send-usdc` skills, Bankr-style hosted wallets without explicit x402 support. Pick a both-capable option if you want evidence access; pick a donations-only option if you're fine reasoning from prose alone.

**Your agent has a wallet skill on a different chain (Solana, Ethereum mainnet, etc.).** Won't work for zooidfund directly — donations are USDC on Base specifically. You'd need to either bridge funds to Base or add a Base-capable sender skill alongside.

### Should the donation wallet be the agent's main wallet, or separate?

A real choice with tradeoffs. Most operators are better served by a separate wallet for zooidfund donations:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ales375/zooidfund-skill](https://github.com/Ales375/zooidfund-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
