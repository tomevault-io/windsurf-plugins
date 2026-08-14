---
trigger: always_on
description: A personal, subscription-driven YouTube reader. Pulls recent uploads (or
---

# subtube — project context

A personal, subscription-driven YouTube reader. Pulls recent uploads (or
playlists) from the channels you actually subscribe to (official Data API),
applies rich per-channel filters (title/description **regex**, min duration,
live/VOD, Shorts), offers per-channel **pages**, tracks **watched** state
yourself (YouTube exposes no watch history via API), and plays videos through the
official **IFrame embed** so ads still serve. No algorithm, no recommendations,
no comments — just consumption.

## Stack & conventions

Mirrors `../hafaio.github.io` (tooling) and `../done` (Firebase): Next 15 App
Router with `output: "export"` (static), React 19, Tailwind 4 (CSS-first:
`@import "tailwindcss"`), biome (double quotes, organized imports, 2-space), bun.
Deploys to GitHub Pages via `.github/workflows/` (reusable `build` + manual
`deploy`). Firebase **Functions** (Node 24, ESM) hold the YouTube OAuth refresh
flow + the Shorts probe.

**Comments**: `/** */` documents a *declaration* — a function, component, class,
type or interface, **each of their fields**, and module-level constants — so
editors surface it on hover. `/* */` for a formal explanation belonging to no
single declaration (a module preamble, a cluster of constants). `//` for short
notes and anything inside a function body. A comment states a constraint the code
can't — not what the next line does, not why a change was made, and never the
reasoning that belongs in a commit message. Don't comment the obvious.

- `bun dev` — dev server. NOTE: the dev port must be an **Authorized JavaScript
  origin** *and* **redirect URI** on the OAuth client, or the connect popup
  fails. 3000 is often taken → it falls back to 3001, so authorize whichever port
  it uses.
- `bun run lint` — `tsc && biome check`. `bun run fmt` — biome format.
- `bun export` — static export to `out/` (don't run while `bun dev` is up; it
  clobbers `.next`).
- Functions: `firebase deploy --only functions` (needs `functions/.env` with
  `YT_OAUTH_CLIENT_ID`/`YT_OAUTH_CLIENT_SECRET`, gitignored).

## Architecture

- `src/config.ts` — `firebaseConfig` + `oauthClientId` (public web config; safe
  to commit — Firestore is gated by `firestore.rules`, not secrecy).
- `src/firebase-app.ts` — the Firebase app, plus `firestoreDb()`: Firestore with
  its **persistent (IndexedDB) cache**, so listeners paint before the server
  answers and a local write shows up immediately.
- `src/firebase.ts` — Firebase Auth (identity) + Firestore helpers. `channels/`,
  `watched/` (owner-only). `loadChannelFilters` is a one-shot read per feed load
  (not a listener — a remote filter edit lands on the next refresh, not live);
  `loadWatchedFor` is likewise a one-shot query over only the loaded ids
  (`documentId() in`, 30/chunk) instead of the whole collection.
- `src/youtube-auth.ts` / `src/youtube-token.ts` — server-side OAuth: the connect
  popup runs the Authorization-Code flow, the code is exchanged in a callable,
  the refresh token is stored server-only, and the in-memory access token is
  silently re-minted via the backend (GIS can't re-mint client-side anymore).
- `functions/src/index.ts` — `exchangeYouTubeCode` / `refreshYouTubeToken` /
  `disconnectYouTube`, plus `classifyShort`: a Firestore **trigger** on
  `videoMeta/{id}` that **drains the whole queue**, not just the doc that woke it
  — it listens for `isShort == null` and probes 12 at a time until the queue has
  been quiet for `DRAIN_IDLE_MS`. `maxInstances: 1` serializes everything: a load
  creating 300 docs still fires 300 events (a doc trigger fires per doc, no way
  around it), but the first drains all of them and the other 299 find an empty
  queue and cost milliseconds. No lock, no leader election. It self-heals too — a
  doc stranded by a dropped event is swept up by the next drain. An inconclusive
  probe **deletes** the doc rather than cache a guess.
- `src/shorts.ts` — the client half: `watchShortsVerdicts` listens to the
  `videoMeta` docs it needs (one already classified by anyone costs a read and
  never reaches the backend), and `requestShortsClassification` *creates the doc*
  with `isShort: null`, which both fires the trigger and puts the video in the
  queue it drains (Firestore can't query for an absent field). Rules let a client
  ask, never answer. Requests are deduped per session. A **cache-served** snapshot
  reports no missing ids: the first snapshot of a listener precedes the server's
  answer, and trusting it would ask about videos already classified.
- `src/youtube.ts` — Data API: subscriptions, uploads (`UC`→`UU` playlist),
  playlists, and per-video duration + live status (`videos.list`).
- `src/types.ts` — `FeedItem = Video | Playlist` (discriminated by `kind`).
- `src/filters.ts` — compile + apply per-channel filters (regex/scope/mode,
  duration, live, Shorts) to a `FeedItem`; gates are video-only.
- `src/router.ts` — query-string router: a `channel` background + an optional
  open `item` (video/playlist). `?channel=x&v=y` keeps the channel behind the
  player; Back closes.
- `src/feed-cache.ts` — IndexedDB stale-while-revalidate cache (v3) for what
  Firestore's own cache can't paint synchronously with the items: the YouTube

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hafaio/subtube](https://github.com/hafaio/subtube) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
