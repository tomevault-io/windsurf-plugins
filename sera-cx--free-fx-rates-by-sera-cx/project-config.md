---
trigger: always_on
description: **Goal:** turn AI agents (and devs) into counted users of the Sera network — with the
---

# Agents — GTM + build spec for agents.sera.cx

**Goal:** turn AI agents (and devs) into counted users of the Sera network — with the
simplest possible onboarding for both. Companion to `HANDOVER.md` (rates layer + Privy auth).

> Target design (endpoints below aren't all live yet). The north star: a dev **or** an agent
> gets from zero to a working rate call in one step, with no forms and no complicated UX.

---

## 1. Design principle: one identity, two front doors, zero forms

- **One identity primitive:** the **wallet** (proven by an EIP-712 signature — the same thing
  Sera already uses for `ManageApiKey`). The wallet address *is* the Sera user.
- **Two thin front doors to the same backend:**
  - **Humans (devs):** click **"Connect with Privy"** → one signature → key appears. No forms.
  - **Agents (machines):** one signed API call → key returned. No UI at all.
- **Registration is implicit.** Minting a key already proves wallet control, so **the wallet is
  registered the moment it gets a key.** There is no mandatory second step. Directory
  enrichment (name, capabilities) is **one optional call**, skippable.

If either party has to think about more than one step, we built it wrong.

---

## 2. Who's calling (two populations — treat differently)

1. **Answer engines / crawlers** (ChatGPT, Perplexity, Gemini, Google AI): read public rates to
   cite you. **Keyless + SEO.** Never "users" — they're distribution. Don't make them do anything.
2. **Transacting agents** (payment agents, copilots, agent frameworks): the ones you want as
   **users**. They need a wallet to act — so they self-identify by getting one. This doc is about them.

---

## 3. The onboarding (side by side — identical underneath)

**Dev (human) — 2 clicks, ~30s**
1. fx.sera.cx dashboard → **Connect with Privy** (email / social / wallet; Privy makes an
   embedded wallet if they have none).
2. Click **Generate key** → sign once (gasless EIP-712) → copy `sera_pk_…` + a snippet. Done.

**Agent (machine) — 1 call, headless**
```bash
# Operator provisions the agent wallet once (Privy server wallet, or bring your own),
# then the agent mints its own read-only key with a single signed call:
POST https://api.sera.cx/api/v1/api-keys      # EIP-712 body, action=create → {api_key, api_secret}
```
Or zero setup for read-only awareness: just call the **keyless** public rate endpoint (below).

Same wallet primitive, same key, same backend. The only difference is Privy renders the click
for humans; agents call the API. **No separate "agent flow" to build or maintain.**

---

## 4. Identity & registration

- **Mint key = register.** After `POST /api-keys`, the wallet is a Registered Sera participant.
  Nothing else required to be counted as a user.
- **Optional directory enrichment (one call, skippable):**
```json
POST https://agents.sera.cx/agents/register        // EIP-712 signed by the same wallet
{
  "name": "Acme Payment Copilot",
  "wallet": "0xabc…",                 // identity — proven by the signature
  "surfaces": ["mcp", "rest"],
  "capabilities": ["quote", "convert", "settle"],
  "domain": "acme.ai"                 // OPTIONAL display label, NOT verified (see §7)
}
```
No `.well-known`, no DNS, no domain proof, no captcha. One signature, done.

---

## 5. Trust tiers (only two)

| Tier | Proof | Gets |
|---|---|---|
| **Registered** | wallet sig + key | counted user; higher rate limits; directory listing; settlement rebates |
| **Anonymous** | none (keyless) | public rates only, low rate limit — awareness / citations |

`localhost` / private agents are just **Registered without a domain label** — real users, nothing special to do.

---

## 6. Incentives (why anyone registers — it's a value exchange, never a gate)

Registration is **required for perks, never for reading rates**:
- Higher rate limits + premium/streaming endpoints.
- **Listing in the public agent directory** (agents.sera.cx) — free discovery for them, an
  SEO/moat asset for us.
- **Settlement fee rebates / revenue share** — the real hook: register + route volume → earn.
- Optional **"Powered by Sera"** attribution (voluntary; our distribution flywheel).

---

## 7. What we're deliberately NOT building (keeps it simple)

- ❌ Domain control / `.well-known` / DNS verification — dropped. `domain` is an unverified label.
  (Re-addable later as an optional "verified ✔" upgrade *because* identity is decoupled from it.)
- ❌ Multi-step signup wizards, captchas, KYC, email verification loops.
- ❌ A separate agent onboarding flow — agents use the same key API humans' Privy click calls.
- ❌ "Prove you're an AI" detection — unsolvable and unnecessary; the wallet + volume is the user.

---

## 8. Build specs

**Auth (all authenticated calls):** `Authorization: Bearer {api_key}:{api_secret}` (read-only key).
**Bases:** rates layer `RATES_BASE` (TBD — see HANDOVER); core API `https://api.sera.cx/api/v1`;
agents `https://agents.sera.cx`.

**Rates (keyless public + keyed higher-limit) — served by the rates layer, CORS-enabled:**
```
GET  {RATES_BASE}/quote?from=EUR&to=USD          → { "buy":1.0851, "mid":1.0842, "sell":1.0833 }
GET  {RATES_BASE}/latest?base=EUR                → { "base":"EUR","asOf":"…","rates":{ "USD":1.0842, … } }

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sera-cx/FREE-FX-Rates-by-Sera.CX](https://github.com/sera-cx/FREE-FX-Rates-by-Sera.CX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
