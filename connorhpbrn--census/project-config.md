---
trigger: always_on
description: Private Telegram money agent. Reads a bank through TrueLayer, keeps a SQLite ledger, answers spend questions.
---

# Census

Private Telegram money agent. Reads a bank through TrueLayer, keeps a SQLite ledger, answers spend questions.

If you are an agent setting this up or changing it: do the work. Do not add a framework, a runtime package, Discord, MCP, Postgres, or a second process unless the operator asks.

## Stack

- Bun 1.4+, TypeScript, **zero runtime npm dependencies**
- `bun:sqlite` on a Railway volume at `/data`
- OpenRouter (`/api/v1/chat/completions` + tools)
- Telegram only
- TrueLayer Data API (live) for the bank

`src/` is the whole app. Read `src/index.ts` then the file you need.

## Commands

```bash
bun install
bunx tsc --noEmit
bun dev          # local, polls Telegram unless PUBLIC_URL is set
bun src/index.ts # production start (see railway.toml)
```

Do not add scripts or deps to make those work.

## Model

Do not hardcode a model. Do not put a model slug in source, README, `.env.example`, or setup replies.

`OPENROUTER_MODEL` is required. Any OpenRouter model that supports tool calling works. The operator picks it. Boot fails if it is missing.

## Setup, local

1. Bun 1.4+.
2. `cp .env.example .env`
3. Fill `TELEGRAM_BOT_TOKEN`, `OPENROUTER_API_KEY`, `OPENROUTER_MODEL`, `ALLOWED_TELEGRAM_USER_ID`.
4. `bun dev`
5. Message the bot. First reply asks currency + city (`GBP London`, or `yes` to accept the Telegram language guess).

Bank connect locally also needs `PUBLIC_URL` as a reachable `https` origin and that exact origin plus `/truelayer` in TrueLayer Console. Without it, the bot still works as a typed ledger.

Never commit `.env`. Never print secrets.

## Setup, Railway

Fork or clone. **New project → Deploy from GitHub**.

Railway will fail until a volume is attached. Mount it at `/data`. **One replica.** SQLite cannot be shared. `railway.toml` already sets `requiredMountPath = "/data"`, `numReplicas = 1`, `overlapSeconds = 0`.

Generate a public domain. Census builds `PUBLIC_URL` from `RAILWAY_PUBLIC_DOMAIN`. **Do not set `PUBLIC_URL` on Railway.**

Required variables (empty until the operator pastes):

| Variable | From |
|---|---|
| `TELEGRAM_BOT_TOKEN` | BotFather `/newbot` |
| `OPENROUTER_API_KEY` | openrouter.ai/keys |
| `OPENROUTER_MODEL` | any OpenRouter model that can call tools. Operator chooses. |
| `ALLOWED_TELEGRAM_USER_ID` | numeric id from @userinfobot. Comma-separated if several. |
| `TRUELAYER_CLIENT_ID` | console.truelayer.com |
| `TRUELAYER_CLIENT_SECRET` | download once from Console |

Optional: `CURRENCY` + `TZ` (skips the place question), `LOCALE`, `TRUELAYER_ENV=sandbox` (Mock Bank only).

After deploy: message the bot → place question → `/connect`.

Telegram: BotFather `/newbot`, optional `/setuserpic` with `logo.png`. The allowlist is the security model. Anyone who finds the bot can message it; only listed ids get a reply.

## TrueLayer

Do not "simplify" this. Revolut will break.

- Live Console for a real bank. Sandbox is Mock Bank only.
- Redirect URI, exact, no trailing slash: `https://<domain>/truelayer`
- Live Data API must be enabled on the app. If the bank never appears, ask TrueLayer to turn Data on for that `client_id`.
- Do not put an access token in env. It dies in about an hour. After `/connect`, the refresh token lives in SQLite on the volume, plaintext. Treat the volume as a secret.
- `TRUELAYER_REFRESH_TOKEN` is an optional seed if they already have a refresh token. Still no access token.
- `/connect` → operator must finish in **5 minutes**. Revolut only lists `/accounts` inside that SCA window. Persist account ids. Later syncs must **not** re-list `/accounts`.
- First sync (inside SCA): history from `2015-01-01`. After that: last 90 days.
- Auth scopes encoded as `%20`, not `+`. Providers: `uk-ob-all ee-ob-all`. See `src/bank.ts`.
- `X-PSU-IP` is the browser IP from the OAuth callback, then reused. Never Telegram's webhook IP.
- Dedup: `normalised_provider_transaction_id`, then `provider_transaction_id`, then `transaction_id`.
- Card DEBIT amounts can be positive. Use `transaction_type`, not sign.
- Credits that look like pay are stored in `incomes`. Transfers, pots, ATM, and refunds are skipped. Do not add credits into spend. Infer take-home from repeating inbound and update `income_monthly` when the bank is clear. Subscriptions are inferred from repeating bank charges, plus anything typed. Do not count a cancelled `merchant_key` again.
- Pending: pull `/transactions/pending` per account. `pending=true` on the row. Wipe that account's pending on each successful pending list so a settled charge cannot double-count. If pending is 403/SCA/501, leave existing pending alone.
- Dead refresh (`invalid_grant`) → clear bank, tell them `/connect`. Do not wipe the ledger on a generic `/accounts` 403.
- 429: retry once.

`/disconnect` drops tokens and account ids. Imported spends stay.

## How the bot is supposed to work

The operator's bank is the source of spend. The agent reads the ledger. It does not invent merchants or totals.

- Money is integer minor units (`amount_pence`). Tools take major units.
- `spend_summary.expenses` / `per_day` / `projected_month` are **home currency only**. `by_currency` is the rest. Do not convert. Do not add EUR into GBP.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [connorhpbrn/census](https://github.com/connorhpbrn/census) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
