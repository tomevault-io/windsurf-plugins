---
trigger: always_on
description: Orientation for Claude Code working in this repo.
---

# CLAUDE.md

Orientation for Claude Code working in this repo.

## What this is

`life-structure` is a single-file progressive web app — a quiet personal system for money, routines, wellness, and supplements. Its purpose is to reduce daily decision fatigue and keep working when the user is stressed, tired, or emotional. Design intent: calm, low-stimulation, non-judgmental. The tone of any copy you write should match: gentle, plain, never shaming.

## Architecture

Almost everything lives in **`life-structure.html`** — HTML, CSS, and JavaScript in one file, no framework, no build step, no external runtime dependencies.

- **CSS** is in a single `<style>` block driven by CSS custom properties in `:root`. Theme is dark with gold (`--gold`), green (`--green`), and blue (`--blue`) accents. Reuse the variables; do not hardcode colors.
- **JS** is one inline `<script>` at the end of `<body>`. It is plain ES (no modules, no bundler).

Supporting files make it an installable PWA:
- `manifest.json` — metadata + icon references.
- `service-worker.js` — network-first cache with offline fallback. Bump `CACHE` when assets change.
- `icons/` — generated PNGs (192, 512, 512-maskable).

## State model

A single object `state` is persisted to `localStorage` under `STORE_KEY = "lifeStructure.v1"`.

```
state = {
  finance:    { income, bills, rate, efTarget, savings, treatLimit },
  treats:     { spent, weekStamp, log:[amt...] },     // log powers Undo
  anchors:    { stamp, done:[bool x4] },
  supps:      { stamp, done:[bool x4] },
  fitness:    { stamp, done:[bool x3] },              // Stretching / Calisthenic strength / Posture (FITNESS array)
  planner:    { Monday:{p,m,w}, ... },                // p=priority, m=money move, w=wellness move
  spendTimer: { endsAt },                             // ms epoch; persists the 20-min wait across reloads
  streaks:    { anchors:{n,last}, supps:{n,last}, fitness:{n,last} }, // consecutive-day completion
  xp:         0,                                      // System XP (see RANKS / levelInfo)
  awards:     { stamp, anchors:[bool x4], supps:[bool x4], fitness:[bool x3] }, // per-day XP-award guard
  invest:     { holdings:[ {id,ticker,shares,cost,price,market,currency} ], history:[ {d,v} ], updated, fx },
                                                             // market: US|JSE; currency: USD|JMD; fx = JMD per 1 USD; history in baseCcy
  property:   { target, saved, monthly, log:[amt...] },      // property fund goal + Undo
  certs:      [ {id,name,target,progress,status,awarded} ],  // status: Planned|In progress|Earned
  baseCcy:    "JMD",                                         // base currency for savings/property/net worth (JMD|USD)
  vitals:     [ {id,d,sys,dia,weight,hr,steps} ],            // BP / weight / resting HR / steps; one record per date (merged)
  view:       "full",                                        // "full" | "minimal" (the Now focus screen)
  schedule:   { supps:{i:"HH:MM"}, fitness:{i:"HH:MM"} },    // time-of-day overrides; defaults live on supplement/FITNESS `.time`
  suppList:   [ {name,dose,schedule,purpose,tags,time,notes} ], // user's own supplements; empty → generic SUPPLEMENTS defaults
  profile:    { considerations:[], allergies:[], avoid:[] }  // personal health profile; ships empty
}

**Privacy rule:** the committed code must stay generic. Personal supplements and health-profile content live only in `state` (`suppList`, `profile`) — never hardcode them back into HTML/JS/docs. `SUPS()` resolves `state.suppList` → generic `SUPPLEMENTS` fallback; always use it (not the constant) when reading the active list. `*.private.json` is gitignored for local personal-data backups.
```

Bump `STORE_KEY` only for breaking schema changes (wipes user data). New keys are merged in `load()` with defaults, so additive changes are safe without a bump.

Key functions:
- `load()` / `save(quiet)` — read/write `localStorage`; `save()` shows a brief "Saved" flash unless `quiet`. `load()` deep-merges defaults for each nested slice so old saves stay valid.
- `applyResets()` — clears daily items (anchors, supps, **awards**) when `stamp !== today()` and treats when `weekStamp !== weekStamp()`. Called on load and on `visibilitychange`.
- `compute()` — money math: `afterBills = income - bills`; `save = afterBills * rate/100`; `flex = afterBills - save`.
- **System:** `xpForLevel(L)`, `levelInfo(xp)` (→ level/into/span/pct/nextAt), `rankFor(L)` (RANKS E→S), `awardXP(n)` (detects level-ups → `showLevelUp`), `renderSystem()`.
- `render*()` — one per UI area: `renderDashboard`, `renderToday`, `renderAnchors`, `renderSupps`, `renderPlanner`, `renderGlance`, `renderLabs`, `renderSystem`, `renderInvest`, `renderProperty`, `renderCerts`, `renderAscSummary`.
- Ascension actions: `addHolding`/`delHolding`/`updHoldingPrice`, `addProperty`/`undoProperty`/`addSuggestedToProperty`/`usePlanAsMonthly`, `addCert`/`updCert`/`earnCert`/`removeCert`. Forms bound in `bindFinance()` and `bindAscension()`.
- `renderNetWorth()` = savings/EF + property fund + portfolio value (Dashboard card). `snapshotPortfolio()` records one portfolio data point per day → `renderSpark()` draws an inline SVG sparkline.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ettalin/life-structure](https://github.com/ettalin/life-structure) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
