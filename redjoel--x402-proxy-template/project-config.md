---
trigger: always_on
description: Context for AI coding agents to help users set up x402-proxy for payment-gated content.
---

# x402-proxy Setup Guide for AI Agents

Context for AI coding agents to help users set up x402-proxy for payment-gated content.

## What is x402-proxy?

A Cloudflare Worker that adds payment gating to any origin using the x402 protocol. Users pay to access protected routes, then get a JWT cookie valid for 1 hour.

**With Bot Management Filtering:** Requires Bot Management for Enterprise to enable bot filtering. With it enabled, x402-proxy can implement "default closed" - humans pass through free, only bots must pay. This is optional and enhances the base functionality.

---

## Interactive Setup Flow

When a user asks for help setting up x402-proxy, follow this discovery process:

### Step 1: Verify Cloudflare Authentication

```bash
npx wrangler whoami
```

If not logged in, guide them to run `npx wrangler login`.

If they have multiple accounts, note them for Step 2.

---

### Step 2: Select Domain

Ask: **"Which domain do you want to add payment gating to?"**

If the user has multiple Cloudflare accounts (from Step 1), also ask: **"Which account is this domain on?"**

**Save the domain** - it scopes everything that follows.

---

### Step 3: Check for Bot Management (Optional Enhancement)

Ask: **"Do you have Bot Management enabled on `{domain}`?"**

Explain why you're asking:

> With Bot Management, x402-proxy can implement "default closed" - blocking bot traffic by score threshold while letting humans through automatically. You can also make specific exceptions for bots like Googlebot or verified AI crawlers.
>
> Without Bot Management, x402-proxy still works perfectly - it just charges for protected routes without distinguishing between bots and humans. All traffic to protected routes must pay.

| Answer  | Effect                                            |
| ------- | ------------------------------------------------- |
| **Yes** | Enable Bot Management Filtering prompts in Step 4 |
| **No**  | Skip threshold/exception prompts in Step 4        |

---

### Step 4: Configure Protected Paths (Iterative)

Ask: **"What path on `{domain}` do you want to charge for?"**

If the user provides multiple paths at once, queue them and configure each in sequence.

**For EACH path, ask:**

#### 4.1 Price

Ask: **"What price (in USD) for `{path}`?"**

Format: `$0.01`, `$0.10`, `$1.00`, etc.

#### 4.2 Description

Ask: **"What description for `{path}`?"** (shown to users explaining what they're paying for)

Example: "Access to premium content for 1 hour"

---

#### If User Has Bot Management (from Step 3):

Continue with these additional prompts:

#### 4.3 Bot Score Threshold

Ask: **"What bot score threshold for `{path}`?"**

**ALWAYS offer exactly these three options:**

| Option               | Threshold | What it means                                                  |
| -------------------- | --------- | -------------------------------------------------------------- |
| **1**                | 1         | Very strict - only verified humans pass free                   |
| **2**                | 2         | Strict - only clear human traffic passes free                  |
| **30 (Recommended)** | 30        | Balanced - likely automated traffic must pay, humans pass free |

**Recommended: 30** - This is the typical starting point that blocks likely-automated traffic while letting humans through free.

#### 4.4 Bot Exceptions

Ask: **"Any bots that should get FREE access to `{path}`?"**

**Offer these preset options:**

| Preset                    | Bots Included                                                            | Use When                     |
| ------------------------- | ------------------------------------------------------------------------ | ---------------------------- |
| **Googlebot + BingBot**   | Googlebot, BingBot                                                       | Allow major crawlers         |
| **Above + AI assistants** | Above + ChatGPT-User, Claude-User, Perplexity-User, Meta-ExternalFetcher | Allow AI assistant citations |
| **None**                  | (empty)                                                                  | All bots must pay            |

If the user selects a preset or names specific bots:

1. Look up each bot name in the Bot Registry (see below)
2. Resolve to detection IDs
3. Write to config with inline comments

**Example resolution:**

- User says: "Googlebot and BingBot"
- Agent looks up: Googlebot → 120623194, BingBot → 117479730
- Config output:

```jsonc
"except_detection_ids": [
  120623194,  // Googlebot
  117479730   // BingBot
]
```

---

#### After Configuring Each Path

Ask: **"Any more paths on `{domain}` to protect?"**

- If **yes** → repeat Step 4 for the next path
- If **no** → continue to Step 5

---

### Step 5: Wallet & Network Configuration

Ask these together:

1. **"What wallet address should receive payments (PAY_TO)?"**
2. **"Which network: `base-sepolia` (testing) or `base` (production)?"**

#### If User Doesn't Have a Wallet Address

They can use the default "dead address" for testing:
`0x000000000000000000000000000000000000dEaD`

For production, they'll need a real wallet:

- [Coinbase Wallet](https://www.coinbase.com/wallet)
- [MetaMask](https://metamask.io)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [redjoel/x402-proxy-template](https://github.com/redjoel/x402-proxy-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
