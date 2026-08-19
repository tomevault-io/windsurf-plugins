---
trigger: always_on
description: You are working on **CashFromChaos**, a hackathon demo/product prototype for the **Hermes Agent Accelerated Business Hackathon** by Nous Research × NVIDIA × Stripe.
---

# CashFromChaos — Claude Code Operating Brief

You are working on **CashFromChaos**, a hackathon demo/product prototype for the **Hermes Agent Accelerated Business Hackathon** by Nous Research × NVIDIA × Stripe.

This is **not** a generic SaaS generator and not merely a Wallapop automation tool.

## One-line product

> **Point your camera at things you don’t want. Hermes sells them.**

CashFromChaos is an autonomous recommerce operator: a user sends photos of a real object they no longer want, gives minimal context, and Hermes handles the selling operation end-to-end: understand the item, ask only critical questions, decide where it should sell, price it, create listings, negotiate with buyers under policy, collect payment with Stripe, guide shipping/pickup, and release payout when the transaction completes.

## Core promise

The user should only need to know:

> “I don’t want this.”

The system should respond:

> “Thanks. Relax, enjoy your life. I’ll tell you when there’s a buyer.”

The seller should do only two things:

1. point at the object / provide photos and minimal missing details;
2. ship it or meet the buyer when Hermes tells them exactly what to do.

Everything else is autonomous operation.

## Product thesis

Most hackathon teams will build agents that generate a SaaS, landing page, or digital business. That is conceptually easy to copy with AI.

CashFromChaos operates on messy real-world inventory:

- unique physical objects;
- imperfect photos;
- missing details;
- category-specific marketplaces;
- uncertain pricing;
- human negotiation;
- scam risk;
- shipping/pickup decisions;
- payment custody and payout;
- operational traceability.

The differentiator is not “AI writes listings”. The differentiator is **policy-bound autonomous commerce over physical inventory**.

## Hackathon judging alignment

CashFromChaos demonstrates agents that:

- **earn**: buyer payment via Stripe;
- **spend**: shipping labels, boosts, packaging, verification, marketplace fees, or fulfillment costs within policy;
- **run real operations**: listing, negotiation, customer support, payment, logistics, delivery confirmation, P&L.

It should be demoable as a working artifact, not just a concept.

## Target demo narrative

Build toward a 1–3 minute demo video.

### Scene 1 — Seller hook

David faces camera:

> “I don’t want this.”

He turns the camera toward a concrete real object and says something minimal:

> “I want to sell these Pokémon cards.”
> “I want to sell this guitar pedal.”
> “I want to sell this kid’s stroller.”
> “I want to sell this piece of furniture.”

The user should give a **small clue** about the target object. Do not make the primary UX “scan an entire room and infer ten things”. That is impressive but less reliable and less clear.

### Scene 2 — Guided intake

The app/Hermes receives photos and analyzes the item.

It should ask only critical missing details, for example:

- exact model / serial number;
- condition;
- whether it works;
- accessories/manual/box;
- dimensions for bulky items;
- pickup vs shipping preferences;
- minimum acceptable price.

UX principle:

> **Minimal human clue + maximal autonomous operation.**

### Scene 3 — Operational thinking

Show a visual dashboard or log with legible decisions, not hidden magic.

Example:

```txt
Item: Pokémon card binder
Category: collectibles / trading cards
Confidence: medium-high

Missing info:
- Need close-ups of rare/holographic cards
- Need language/edition if visible

Market strategy:
- Avoid local marketplace first
- Better expected demand in collector channels
- Bundle listing recommended unless rare cards found

Target price: €90
Floor price: €65
Autonomy: counteroffer down to €75
Human approval required below €65
Fulfillment: tracked shipping
Max spend: €8
```

### Scene 4 — Marketplace routing

The system is **marketplace-agnostic**. It should choose where to sell based on the item, not default to Wallapop.

Examples:

- Pokémon cards → collector marketplace / Cardmarket-like marketplace / specialist forum / eBay fallback.
- Instrument/music gear → Reverb-like channel / Wallapop / eBay.
- Furniture → local pickup only, Wallapop/Facebook Marketplace-like channel.
- Children’s items → local marketplaces / parent groups / Vinted if clothing.
- Generic electronics → Wallapop first, eBay fallback if rare.

For the hackathon MVP, real adapters are optional. A marketplace sandbox is acceptable if the app clearly models adapters and actions.

### Scene 5 — Buyer persona

Demo can use David with a fake moustache as the buyer.

Buyer opens the marketplace/sandbox and messages the listing:

> “Would you take €50?”

Hermes negotiates under policy:

```txt
Buyer offer: €50
Floor: €65
Decision: reject and counter at €75
Reason: below seller floor and market comps support higher price.
```

Response:

> “I can do €75 if you pay today. Tracked shipping included / pickup available.”

### Scene 6 — Stripe payment and custody

Buyer pays with Stripe.

Use language carefully: implement or simulate an **escrow-like marketplace flow**; do not overclaim legal escrow unless actually compliant.

Acceptable wording:

- “Stripe-powered held payment”;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DavidDiazMerino/cashfromchaos](https://github.com/DavidDiazMerino/cashfromchaos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
