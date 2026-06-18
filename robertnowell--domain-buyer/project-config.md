---
trigger: always_on
description: Purchase a domain name via the Namecheap API with a hard spending cap and explicit human confirmation. Use when the user says "buy <domain>", "register <domain>", "purchase <domain>", asks Claude to acquire a domain, or wants to check availability and price before buying. Drives the full flow inline in the conversation — Claude asks questions, runs the API calls, confirms with the user before any spend.
---


# Domain Buyer

In-Claude domain registration. Claude orchestrates setup, availability checks, and purchase via the Python wrapper at `~/.claude/skills/domain-buyer/domain_buyer.py`. No standalone CLI wizard — every prompt happens in chat.

## Architecture

- `domain_buyer.py status` — JSON state probe (config exists? sandbox? cap? current public IP?)
- `domain_buyer.py check <domain> --json` — availability + price + over_cap flag, JSON
- `domain_buyer.py buy <domain> --confirmed --json` — registration, JSON result. `--confirmed` tells the script that Claude has already confirmed with the user; without it, the script falls back to a TTY prompt for direct-CLI fallback use.
- `domain_buyer.py log --json` — purchase history, JSON

Config lives at `~/.config/domain-buyer/config.json` (chmod 600). Claude reads and writes this file directly with the Read/Write tools. The Python script never prompts.

## Safety invariants (non-negotiable)

1. **One domain per `buy` call.** Never batch. If the user wants three domains, do three independent confirmations.
2. **Per-purchase cap.** Default $50 USD, lives in config. Enforced via the over-cap second-confirmation flow, not via a hard refusal.
3. **Type-the-domain confirmation.** Before invoking `buy`, Claude must ask the user to type the exact domain name in chat. Anything other than an exact match aborts. If over cap, additionally require the user to type `override` after the domain confirmation.
4. **Re-check immediately before buying.** Run `check` again right before `buy` so the price you confirmed against is the price that gets charged. No stale data.
5. **Never invent prices or API responses.** Every dollar figure shown to the user must come from the script's JSON output.

## Pre-flight (always run first)

When invoked for any reason, start with:

```bash
python3 ~/.claude/skills/domain-buyer/domain_buyer.py status
```

Parse the JSON. Branch:
- `config_exists: false` → run the **Setup flow** below.
- `config_exists: true` → run the **Check / Buy / Log flow** the user asked for.

The `status` output also includes `current_public_ip` — useful when the user's IP changed since setup.

## Setup flow (run when no config)

### Step 1 — Tell the user what's coming

In chat, explain in 2-3 sentences: we need API access from Namecheap (with the IP-whitelist prerequisite), registrant contact info (ICANN requirement), and a spending cap. Note that the API key will appear in chat transcript — they should rotate it after setup if that's a concern.

### Step 2 — Sandbox vs production + cap (AskUserQuestion)

Ask via `AskUserQuestion`:
- Q1 "Environment for first setup": `Sandbox (test, free, separate account at sandbox.namecheap.com)` / `Production (real domains, real money)`. Default to Sandbox if they're new to the API.
- Q2 "Per-purchase cap in USD": offer `$50 (default)` / `$25 (cautious)` / `$100 (room for premium .ai/.io)`. Other lets them type a number.

### Step 3 — Credentials (chat free-text)

Tell the user where to get the key: `namecheap.com → Profile → Tools → Namecheap API Access` (or sandbox.namecheap.com for sandbox). Ask them to reply with two values in any reasonable format:
- Namecheap username (also their API user)
- API key

Parse their reply. If ambiguous, ask for the missing piece.

### Step 4 — IP whitelist reminder

From `status`, you already have `current_public_ip`. Tell the user the exact IP and the exact URL to whitelist it:
- Prod: https://ap.www.namecheap.com/settings/tools/apiaccess/
- Sandbox: https://ap.www.sandbox.namecheap.com/settings/tools/apiaccess/

Wait for the user to confirm they've added it before proceeding. (10-min propagation per Namecheap; usually instant.)

### Step 5 — Registrant contact (pull from Namecheap, do NOT re-ask)

Namecheap already stores contact profiles on the user's account. Never make the user re-type information the registrar has on file. Pull it via the API:

```bash
# Lists address profiles (returns AddressId + nickname per profile)
python3 ~/.claude/skills/domain-buyer/domain_buyer.py addresses --json

# If multiple profiles, show the list with nicknames and let the user pick
# If one profile, use it directly after a one-line confirmation
# If zero profiles, fall back to asking the user (see fallback below)
```

Show the retrieved contact info in chat and confirm with the user: *"I'll register domains under this contact — looks right?"* Single AskUserQuestion with options `Yes, use this` / `Use a different saved address` / `Enter manually`.

**Fallback** (no saved address on Namecheap, or user wants to override): ask for one paste block — First name, Last name, Street, City, State/Province, Postal code, Country (e.g. US), Phone (`+1.5551234567` format), Email. Parse, ask targeted follow-ups for missing fields.

### Step 6 — Write the config

Use the Write tool to create `~/.config/domain-buyer/config.json`:

```json
{
  "api_user": "<username>",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robertnowell/domain-buyer](https://github.com/robertnowell/domain-buyer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
