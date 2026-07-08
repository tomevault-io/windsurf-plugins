---
trigger: always_on
description: > Read this before touching any of: `index.html`, `n8n/*.js`, the Tampermonkey
---

# LP2P — Project Conventions & Gotchas

> Read this before touching any of: `index.html`, `n8n/*.js`, the Tampermonkey
> userscript, the `Sourcing Discord` sheet, or the matching algorithm.

The dashboard is a **single static `index.html`** served via GitHub Pages
(`https://slowz942.github.io/DASCHBOARD-RESA-LP2P/`) from `main`. There is
no build step. Edit and push to `main` to deploy. Pages rebuilds in ~30s.

The repo also contains:
- `userscript/discord-wts-scraper.user.js` — Tampermonkey userscript that
  scrapes WTS posts from Discord
- `n8n/*.js` — Code-node bodies for two n8n flows
- `n8n/PHASE-4-SETUP.md` — step-by-step setup of the Telegram flow
- `n8n/SETUP.md` — step-by-step setup of the original WTS-parsing flow
- `google-apps-script/Code.gs` — Apps Script web app that returns inventory
  rows + cell colors as JSON
- `SOURCING.md` — auto-sourcing setup walkthrough

---

## The full pipeline

```
Tally form (client request)              Discord #wts (community sellers)
       │                                            │
       │ Gmail trigger                              │ Tampermonkey userscript
       ▼                                            │ (read-only, no Discord API)
[n8n flow #1: Tally→CRM→Telegram notif]             │ POST to webhook
       │                                            ▼
       ├─ format message (parse Tally email)     [n8n flow #3: Discord→Sheet]
       ├─ Find matches (NEW Code node)              │
       │  ▸ reads Inventory + Sourcing Discord     │ Webhook trigger
       │  ▸ runs the same matcher as the dashboard │ Code: Anthropic claude-3-5-haiku
       │  ▸ POSTs Telegram notification with        │     parses raw WTS message into
       │    inline keyboard of N proposable         │     structured rows
       │    options (calls api.telegram.org         │ Google Sheets append/update
       │    directly, NOT the n8n Telegram node)    │     to "Sourcing Discord" sheet
       └─ append to "CRM clients lp2p" sheet               │
                                                            ▼
[n8n flow #2: Telegram callback handler]    [Google Sheet "Sourcing Discord"]
       │                                            ▲
       ├─ Telegram Trigger (callback_query only)    │
       ├─ branches in PARALLEL:                     │
       │  ▸ answerCallbackQuery (fires            [Dashboard reads via gviz CSV]
       │    immediately — 15s deadline)
       │  ▸ Resolve match (Code) → fetches sheets,
       │    re-runs matcher, picks chosen by index,
       │    builds proposal text → sendMessage
```

### Sheet IDs (already filled in everywhere)
- **Inventory (your stock):** `1jSQNoni7qW6ShnRw3hi_g_fF90qn5YZ3koRaL1gYTLE`
- **Sourcing Discord:** `10QzZ14S4fA5zuM-UyROwmsIKLcwPata6cVLN23bukW8`
- **CRM clients lp2p (Tally writes here):** see n8n config

### Apps Script for inventory colors
URL: `https://script.google.com/macros/s/AKfycbwKCiudNgJU4RtPk-tCv5A33IX3TVtIEJAU_LwbmdhpXHPbWRqYoLbYDUWzkR12zkQ8Hw/exec`

Stored in the user's localStorage on the dashboard (key `lp2p_apps_script_url`).
Hardcoded in n8n Code nodes (constant `APPS_SCRIPT_URL`).

### Telegram bot
- Bot token: stored in n8n Code node (constant `TELEGRAM_BOT_TOKEN` at line ~27 of `find-matches-and-notify.js`). The user pastes their token; **never push a real token to git**. The repo file holds a placeholder.
- Chat ID: `5135913166` (operator's chat with the bot)

---

## Critical conventions (BREAK ONE → BUG)

### 1. Per-place pricing is the source of truth
Inventory's `Achat`/`Revente` columns store **TOTALS** for the whole listing
(verified against the Benef column on actual rows like Roland Garros duo
cat 2 = 490/1400/910 → Benef = 1400 - 490, so totals).

Discord listings carry `price_per_unit` (per-place) from the LLM.

The matcher requires uniform per-place semantics. Do NOT mix totals and
per-place in `prixAchat`. Always run inventory rows through
**`inventoryTotalsToPerPlace(it)`** right after `parseInvRow()`. This
divides `prixAchat` and `prixVente` by `qty` (when qty > 1).

This conversion lives in three files — keep them in sync:
- `index.html` syncInventory (Apps Script + CSV branches)
- `n8n/find-matches-and-notify.js` fetchInventoryViaAppsScript +
  fetchInventoryViaCSV
- `n8n/callback-handler.js` fetchInventory

### 2. Demand totals use BUYER's number of places, not seller's qty
A seller might have a Trio (qty=3) but the buyer wants 2. The proposal
total is `pricePerPlace × buyerPlaces`, not `pricePerPlace × sellerQty`.
Both the dashboard and Telegram use:
```js
const buyerPlaces = parseInt((demand.places||'1').toString().replace(/[^\d]/g,''))||1;
```

### 3. `normalizeArtist` is the canonical entry point
Always pipe artist values through `normalizeArtist()` before storing on a
demand or matching against inventory. The function:
- Strips diacritics (`Céline` → `CELINE`)
- Maps aliases (`CELINE` → `CELINE DION`, `BRUNO` → `BRUNO MARS`, `AYA` →
  `AYA NAKAMURA`, etc.)

Three places have a copy. Keep them in sync. New artists go in the
`ALIASES` literal in all three files.

The dashboard's `syncTelegram()` had a bug where it ran its own
artist-detection regex but never piped the result through
`normalizeArtist()`. Make sure that line is preserved:
```js

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Slowz942/DASCHBOARD-RESA-LP2P](https://github.com/Slowz942/DASCHBOARD-RESA-LP2P) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
