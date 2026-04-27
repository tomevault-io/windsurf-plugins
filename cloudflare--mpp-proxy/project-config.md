---
trigger: always_on
description: Context for AI coding agents helping users set up this MPP payment-gated proxy template.
---

# mpp-proxy Setup Guide for AI Agents

Context for AI coding agents helping users set up this MPP payment-gated proxy template.

## What is mpp-proxy?

A Cloudflare Worker that adds payment gating to any origin using the Machine Payments Protocol (MPP).

- Protected routes return `402 Payment Required` with `WWW-Authenticate: Payment`
- Clients retry with `Authorization: Payment`
- Successful responses include `Payment-Receipt`
- The proxy also issues a JWT cookie valid for 1 hour so browsers and agents can reuse access without paying again on every request

**With Bot Management Filtering:** Requires Bot Management for Enterprise to enable bot filtering. When enabled, humans can pass through free while likely automated traffic must pay. This is optional and enhances the base functionality.

This template is currently wired to the Tempo payment method via `mppx`.

---

## Interactive Setup Flow

When a user asks for help setting up the proxy, follow this discovery process.

### Step 1: Verify Cloudflare Authentication

```bash
npx wrangler whoami
```

If not logged in, guide them to run `npx wrangler login`.

If they have multiple accounts, note them for Step 2.

---

### Step 2: Select Domain

Ask: **"Which domain do you want to add payment gating to?"**

If the user has multiple Cloudflare accounts, also ask: **"Which account is this domain on?"**

Save the domain. It scopes the rest of setup.

---

### Step 3: Check for Bot Management (Optional Enhancement)

Ask: **"Do you have Bot Management enabled on `{domain}`?"**

Explain why:

> With Bot Management enabled, the proxy can implement a default-closed model for bots: likely human traffic passes free, while likely automated traffic must pay. You can also allow specific bots like Googlebot or verified AI assistants through for free.
>
> Without Bot Management, the proxy still works perfectly. All traffic to protected routes must pay.

| Answer  | Effect                                     |
| ------- | ------------------------------------------ |
| **Yes** | Enable Bot Management prompts in Step 4    |
| **No**  | Skip threshold/exception prompts in Step 4 |

---

### Step 4: Configure Protected Paths (Iterative)

Ask: **"What path on `{domain}` do you want to charge for?"**

If the user provides multiple paths at once, queue them and configure each in sequence.

For each path, ask:

#### 4.1 Amount

Ask: **"What amount should `{path}` cost?"**

Format: `0.01`, `0.10`, `1.00`, etc.

Important: this template stores the value as `amount`, not `price`.

#### 4.2 Description

Ask: **"What description should users see for `{path}`?"**

Example: `Access to premium content for 1 hour`

#### 4.3 If Bot Management is enabled

Ask: **"What bot score threshold should `{path}` use?"**

Offer exactly these three options:

| Option               | Threshold | Meaning                                                        |
| -------------------- | --------- | -------------------------------------------------------------- |
| **1**                | 1         | Very strict - only verified humans pass free                   |
| **2**                | 2         | Strict - only clear human traffic passes free                  |
| **30 (Recommended)** | 30        | Balanced - likely automated traffic must pay, humans pass free |

Then ask: **"Any bots that should get FREE access to `{path}`?"**

Offer these presets:

| Preset                    | Bots Included                                                            | Use When                     |
| ------------------------- | ------------------------------------------------------------------------ | ---------------------------- |
| **Googlebot + BingBot**   | Googlebot, BingBot                                                       | Allow major crawlers         |
| **Above + AI assistants** | Above + ChatGPT-User, Claude-User, Perplexity-User, Meta-ExternalFetcher | Allow AI assistant citations |
| **None**                  | (empty)                                                                  | All bots must pay            |

If the user selects a preset or specific bot names, resolve them using the Bot Registry below and write the resulting IDs to `except_detection_ids`.

Example:

```jsonc
"except_detection_ids": [
  120623194, // Googlebot
  117479730  // BingBot
]
```

#### 4.4 More paths

Ask: **"Any more paths on `{domain}` to protect?"**

- If yes, repeat Step 4
- If no, continue

---

### Step 5: Wallet and Tempo Configuration

Ask these together:

1. **"What wallet address should receive payments (`PAY_TO`)?"**
2. **"Use Tempo testnet or production?"**

Map the answer to config like this:

| User Choice | `TEMPO_TESTNET` |
| ----------- | --------------- |
| Testnet     | `true`          |
| Production  | `false`         |

Then ask:

3. **"Which token address should clients pay in (`PAYMENT_CURRENCY`)?"**

Recommended default for demos:

```text
0x20c000000000000000000000b9537d11c60e8b50
```

That is the default PathUSD address used by this template for Tempo testnet demos.

If the user does not have a wallet address yet, they can use the default dead address for smoke testing:

```text
0x000000000000000000000000000000000000dEaD
```

---

## Deployment Phase


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudflare/mpp-proxy](https://github.com/cloudflare/mpp-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
