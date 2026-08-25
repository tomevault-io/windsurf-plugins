---
trigger: always_on
description: A Next.js 15 single-page global situational-awareness map. **Product name: OpenData.**
---

# CLAUDE.md — OpenData (repo `TrafficNerd-V2`)

A Next.js 15 single-page global situational-awareness map. **Product name: OpenData.**
**Prod domain: `traffic-nerd-v2.vercel.app`** — that is the only domain we ship on.
Deployed product = `origin/main`.

## Licence — `AGPL-3.0-only`

This repo is **AGPL-3.0-only** (`LICENSE`, verbatim FSF text; `BRAND.license` in
`lib/brand.ts` is the single source of truth for anything user-facing).

**§13 is an obligation on the deployment, not just a file.** Anyone interacting with
this program over a network must be offered its Corresponding Source, so the console
header (`components/terminal/TerminalHeader.tsx`) and the site footer
(`app/(site)/page.tsx`) both link the repo. **Do not remove those links** — a hosted
AGPL app that does not offer its source is in breach of its own licence.

The AGPL covers **this codebase only**. Every upstream feed keeps its own terms; the
in-app attributions (TfL, Windy, CARTO/OSM, NASA, GDACS, TeleGeography…) are separate
obligations and are not satisfied by the licence.

> **Naming guard.** `worldmonitor.app` / "World Monitor" is a **competitor**
> (`koala73/worldmonitor`, **AGPL-3.0**), not us. Never write it as our domain, our
> product name, or in user-visible strings — **their README reserves branding rights,
> and that is a trademark matter which our licence choice does not touch.**
>
> **What DID change (2026-08-13):** this repo is now AGPL-3.0 itself, so the old
> reasoning here — "lifting their code would force us to relicense" — no longer bites,
> because we have already relicensed and published. Their code is now licence-
> *compatible* with ours. That removes the legal barrier; it does not remove the
> others, and the standing instruction is unchanged: **read their repo for facts only**
> (endpoint URLs, cadences). Copying still requires preserving their copyright notices
> and attribution, and it is a bad *product* call regardless — "we have seen hundreds of
> these, all based on the same one project" is the main thing this project has to
> overcome. If you ever do want to lift something, raise it with Sampo first; it is his
> call and worth a lawyer's glance, not mine.
>
> `simplifaisoul/osiris` is **MIT** and may be copied **with** an attribution header.
> Nothing has been copied from it to date (grep for "Adapted from OSIRIS" returns none).
>
> The two known leftovers this section used to list (`lib/export.ts` naming downloads
> `worldmonitor-*`, and `lib/events/alerting.ts` sending `"World Monitor — Disasters &
> Events"` as an alert source) were **fixed in `18a9de8`**.
>
> Verified 2026-08-11: `grep -rn "worldmonitor\|World Monitor" lib/ app/ components/`
> returns 4 hits and **all four are comments** naming the competitor as a fact — in
> `i18n/catalog.ts`, `monitors.ts`, `sources/keyRequirements.ts` and `api/og/route.tsx`
> (that last one documents the literal it replaced). Those are allowed. What is banned is
> the name in a **user-visible string**, and there are none. Expect the grep to be noisy;
> read each hit before "fixing" it.

## Build gate
- Roadmap: `ROADMAP.md` (driven by the `/goal` milestone loop — one gated milestone per invocation)
- Gate: `npx tsc --noEmit && npm test`   (full check: `npm run build`)
- UI evidence: Playwright screenshots to `persona-shots/`
- Commit: one commit per milestone, `M<n>: <name>`, **solo attribution** (matches every existing commit — no co-author trailer)
- PR: fresh branch + PR per milestone/group. Sampo live-merges and deletes branches fast → always branch off the latest `main` and open a new PR for follow-ons.

## Shape
- **`/` is the marketing site, `/app` is the console.** `app/(site)/` holds the landing
  page (its own layout loads the three marketing typefaces so `/app` never downloads
  them); `app/(console)/app/` holds the shell. `/` forwards any request carrying `?v=`
  or `?c=` to `/app` with the query intact — shared links and OG cards were minted
  against `/`, so removing that shim breaks every link anyone has already sent.
- `components/marketing/*` — landing page only. ONE scroll subscriber
  (`ScrollGround.tsx`) publishes CSS custom properties; nothing else may add a scroll
  listener and nothing may set React state per frame. `.pv-*` tokens in
  `app/provenance.css`, scoped to `.pv-root` so they cannot reach the console.
- The hero is a full-bleed **night stage** (`HeroStage` → `HeroGlobe`), not a plate
  beside the copy. The ground ramp runs **night → day → night**: `--pv-g` starts at
  1, lifts across the hero, and plunges back behind the Adapter panel. `.pv-night`
  is server-rendered in `(site)/layout.tsx` so a cold load does not flash daylight.
  The sticky bar reads `--pv-bar-g` (the ground *directly beneath it*), not `--pv-g`
  — it is the one element that can straddle two grounds at once.
- The hero globe renders **every registered signal layer**, drained into three
  aggregated MapLibre sources (points / lines / fills) exactly like `WorldMap`. The
  layer list is read from `SOURCE_CATALOG` in the server component and passed down
  as a prop — never imported into the client, or all ~39 adapters land in the
  browser bundle. Adding an adapter adds it to the globe with no edit to the hero.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [011-sam-110/Provenance](https://github.com/011-sam-110/Provenance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
