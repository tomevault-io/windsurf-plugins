---
trigger: always_on
description: Hobby project for a fantasy draft built around a tabletop CCG's ("Redemption")
---

# FantasyDraft

Hobby project for a fantasy draft built around a tabletop CCG's ("Redemption")
annual Nationals tournament. Players are drafted onto fantasy teams before
Nationals, then scored on their real tournament performance. The frontend is
a static site (no build step) deployed on GitHub Pages. There is now also a
small Cloudflare Worker backend (`worker/`) powering the live draft feature —
see "Live Draft system" below. Everything else (scouting, standings) stays
static/read-only with no backend of its own.

Live site: https://jhendrix6426.github.io/FantasyDraft/
Repo: https://github.com/jhendrix6426/FantasyDraft — pushing to `main` auto-deploys.

## Structure

- `index.html` — landing page: header banner + tab bar (GM Tools, Commissioner
  Tools, Live Draft, Live Scoring, Draft History, Records). "GM Tools" and
  "Commissioner Tools" are `.tab-group`s — click opens a `.tab-dropdown`
  (GM Tools: Scouting, My Draft Board, My Account, Join Live Draft;
  Commissioner Tools: Live Draft Setup, Scorekeeping) rather than loading a
  page directly; each toggle button gets `.active` whenever any of its own
  sub-tabs is loaded, same visual treatment as a normal tab. The remaining
  four (Live Draft, Live Scoring, Draft History, Records) are plain buttons.
  Tab-switching JS lives inline at the bottom of the file — clicking any
  `[data-tab]` button swaps `#tab-frame`'s `src` to `data-src` if the button
  has one, else `data-tab + '.html'`. `data-src` exists for buttons that
  don't map 1:1 onto `<data-tab>.html`: **Live Draft**/**Live Scoring** (top
  level) point at `draft-presentation.html?year=…`/`scoring-presentation.html?year=…`
  — the read-only broadcast views, which already render their own "waiting"
  placeholder when nothing's live yet, so no separate not-available handling
  was needed here — while **Join Live Draft** (GM Tools) and **Live Draft
  Setup** (Commissioner Tools) both point at `live-draft.html` with
  `?mode=gm` / `?mode=commish` to skip straight past its landing role-picker
  into the right login form (see `landingMode` in `live-draft.html`, below).
  **Scorekeeping** just points at plain `live-scoring.html`, since its
  landing view is already the scorekeeper login with no picker to skip.
  Bump the two hardcoded `year=` values here every season alongside
  `FANTASY_DRAFT_YEAR` in `draft-history.html`/`scouting.html`/`records.html`.
  Adding another plain top-level tab is a new button with a matching
  `data-tab` (+ `data-src` if it needs one); adding another dropdown sub-tab
  is a new button inside the relevant `#…-dropdown` plus adding its
  `data-tab` value to that dropdown's `…TabNames` array (`gmToolsTabNames` /
  `commishToolsTabNames`) so its toggle highlights correctly. All dropdowns
  share one open/close implementation (`.tab-toggle`/`.tab-dropdown` classes,
  not per-dropdown ids) — opening one closes any other that's open. Every
  dropdown closes on an outside click *or* on a click into `#tab-frame` — the
  latter needs its own handling (`window`'s `blur` event) since a click
  inside an iframe never bubbles to the parent document, so a plain
  document-level click listener alone can't catch it. This header only wraps
  a page when it's reached *through* the tab bar (i.e. loaded inside
  `#tab-frame`) — GMs reach `live-draft.html`/`my-board.html` via a direct
  shareable link instead, which bypasses `index.html` entirely. Those pages
  (plus `live-scoring.html`, which can also be opened directly) detect this
  with `window.self === window.top` and render their own small `.site-nav`
  back-link row when true, so a GM arriving cold via their link isn't
  stranded with no way to reach the rest of the site. Don't add this to
  `draft-presentation.html`/`scoring-presentation.html` — those are
  deliberately chrome-free full-bleed broadcast views, always opened
  directly, never wrapped by `index.html`.
- `scouting.html` — the scouting tool. Self-contained single file (HTML/CSS/JS,
  no dependencies, no build). This is where most of the historical-stats work
  has happened.
- `records.js` — shared IIFE (`window.RecordsBook`) for all-time record-book
  computation (format/day high scores, win streaks, draft steals, etc.),
  loaded by `records.html`, `draft-history.html`, and `scouting.html` so the
  logic isn't triplicated. Takes the raw `nationals-history` `db` payload
  directly via `RecordsBook.compute(db)` — see its header comment for the
  full exported API.
- `records.html` — all-time record book UI, reads `records.js`.
- `draft-history.html` — past fantasy drafts (standings, draft order, per-year
  rosters), reads the same `nationals-history` API plus `records.js` for
  badges.
- `live-draft.html` — GM- and commissioner-facing live draft tool (see below).
- `my-board.html` — standalone GM-only view of a single GM's private draft
  board (see "GM draft boards" below), for keeping it open on its own
  tab/device separate from both `live-draft.html` and `scouting.html`. Not
  part of the tab system — opened directly by URL or via a link from
  `live-draft.html`'s board panel.
- `my-account.html` — standalone GM-only page for changing your own password
  (see "Email+password" below). Same not-part-of-the-tab-system status as
  `my-board.html`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jhendrix6426/FantasyDraft](https://github.com/jhendrix6426/FantasyDraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
