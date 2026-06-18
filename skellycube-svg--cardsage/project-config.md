---
trigger: always_on
description: **FeeWorth** is an annual fee renewal decision engine for individuals and couples, built as a Progressive Web App (PWA).
---

# CLAUDE.md — FeeWorth

## Project Overview

**FeeWorth** is an annual fee renewal decision engine for individuals and couples, built as a Progressive Web App (PWA).

- **URL**: https://cardsage.co (domain pending migration to feeworth.co)
- **Tagline**: Is the fee worth it?
- **Audience**: Credit card holders wondering whether to keep, cancel, or downgrade cards at renewal time. Individuals and couples.
- **Contact**: cardsage.co@gmail.com
- **Revenue model**: Affiliate commissions via Apply Now links (CJ Affiliate / FlexOffers). FTC disclosure required — see Affiliate Links section below.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| UI framework | React 18 (via CDN, `react.development.js`) |
| JSX transpilation | Babel Standalone (via CDN, `@babel/standalone`) |
| Build system | **None** — single-file HTML app, no npm, no bundler |
| Fonts | Google Fonts: Playfair Display (serif display / wordmark), Inter (body), Source Code Pro (mono) |
| Analytics | Plausible (`data-domain="YOUR_DOMAIN"` — replace with `cardsage.co`) |
| Hosting | Netlify (auto-deploy from GitHub) |
| PWA | manifest.json + sw.js service worker |

**Key constraint**: Everything runs in the browser. No server, no backend, no database. All state lives in `localStorage`.

---

## File Structure

```
FeeWorth/
├── index.html             # Slim HTML shell (~60 lines) — loads all other files
├── config.js              # Central configuration (single source of truth)
├── styles.css             # All CSS styles
├── firebase-auth.js       # Firebase initialization (module script)
├── cards-data.js          # All data: CARDS, STRATS, TIPS_DB, APPLY_URLS, etc.
├── components.js          # All React components (loaded via Babel Standalone)
├── sw-register.js         # Service worker registration + version check
├── sw.js                  # Service worker (network-first for code, cache-first for assets)
├── version.json           # Deployment version (must match CS_CONFIG.CACHE_VERSION)
├── manifest.json          # PWA manifest
├── icon-192.png           # PWA icon (192×192)
├── icon-512.png           # PWA icon (512×512)
├── privacy-policy.html    # Privacy policy page
├── terms.html             # Terms of service page
├── affiliate-disclosure.html  # FTC affiliate disclosure page
└── .gitignore
```

**Load order in index.html**:
1. `config.js` — central config, injects CSS custom properties into `:root`
2. `styles.css` — all styles (references CSS vars from config.js)
3. `firebase-auth.js` — module script, loads Firebase SDK async, dispatches `cs-firebase-ready`
4. React 18 + ReactDOM 18 + Babel Standalone (CDN)
5. `cards-data.js` — all card/tip/strategy data as globals
6. `components.js` — all React components (`<script type="text/babel">`, transpiled by Babel on DOMContentLoaded)
7. `sw-register.js` — registers service worker + checks version.json for updates

---

## Key Data Structures (cards-data.js)

### `CARDS` — array of ~100 card objects
```js
{
  id: "csr",                      // unique kebab-case ID
  name: "Chase Sapphire Reserve", // full name
  short: "Sapphire Reserve",      // short display name
  issuer: "Chase",
  isBiz: false,
  fee: 550,                       // annual fee in dollars
  network: "Visa",
  cur: "Chase Ultimate Rewards",  // points currency
  c1: "#1a1a2e", c2: "#4a3728",  // gradient colors for card art
  partners: ["Hyatt", "United"],  // transfer partners
  annual: [{n, v, d, cat}],       // annual benefits {name, value, desc, category}
  monthly: [{n, v, d, cat}],      // monthly benefits
  strat: ["chase-trifecta"],      // strategy IDs this card belongs to
  signup: "60,000 pts after $4k in 3 mo",
  earn: {d, g, gas, t, s, a, tr, p, o} // earn rates by category key
}
```

**BENEFITS DATA RULE**: Every recurring benefit must be stored as ONE entry with the correct `reset` field (`monthly` / `quarterly` / `semi-annual` / `annual`). NEVER split a recurring benefit into multiple entries (e.g. "1st half" and "2nd half"). The UI renders multiple checkboxes from a single entry based on the `reset` field. Duplicating entries causes the benefit to appear multiple times in the list.

**SKIP PERSISTENCE RULE**: Skipped benefit state (`cs_skipped`) does NOT reset with quarterly/annual resets — it persists until the user manually un-skips. Skipped benefits are excluded from progress bar calculations (both numerator and denominator) but remain visible in the list under a "SKIPPED" divider at the bottom of each card.

### `STRATS` — object keyed by strategy ID
```js
{
  "chase-trifecta": {
    id, name, emoji, req: ["csr","csp","cfu"],  // required card IDs
    alt: [["csr","cff","cfu"]],                 // alternative card combos
    req_names, desc, forBeginners, analogy,
    firstStep, value, play: [...], learn
  }
}
```
**6 strategies**: `chase-trifecta`, `amex-trifecta`, `c1-duo`, `citi-duo`, `ink-trio`, `atmos-strategy`

### `TIPS_DB` — array of 25 tip objects
```js
{
  id: "t1",
  cat: "sweetspot",     // sweetspot | routing | stacking | timing | arbitrage | application
  title: "...",
  cards: ["csr","hyatt"],  // card IDs relevant to this tip
  body: "...",
  beginnerTip: "...",   // plain-English explanation for beginners

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skellycube-svg/cardsage](https://github.com/skellycube-svg/cardsage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
