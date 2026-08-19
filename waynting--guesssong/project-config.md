---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Longer-form reference lives in [`docs/`](docs/README.md): [architecture](docs/architecture.md) (the system in diagrams), [viral-loop](docs/viral-loop.md) (the loop and how to read `npm run stats`), [operations](docs/operations.md) (deploys and what to do when something breaks), and [decisions](docs/decisions.md) (why it is like this, and what was rejected). This file stays the place for invariants and hazards — rules you must not undo. `docs/` is the place for explanation.

## Commands

```bash
npm run dev        # Start dev server on port 8000 (http://127.0.0.1:8000)
npm run build      # Production build
npm run start      # Start production server
npm run lint       # Run ESLint
npm test           # Run vitest suite (tests/)
npm run stats      # Print the viral-loop counters (needs the Upstash env vars)
```

Use `127.0.0.1:8000` (not `localhost`) — the Spotify app is configured for this origin.

**Run `npm run stats` at the start of any session about growth, the loop, retention, or "what should we build next", and lead with what it says.** Not a nicety. The product's own telemetry went unread for eight weeks across four separate attempts to go and open GA4, and every feature decision in that period was made on an n of 1. The counters exist so that question has an answer; a command nobody runs is the same failure with a shorter path. If the Upstash variables are missing, say so and ask for them rather than reasoning from guesses.

**Read `docs/viral-loop.md` before interpreting the output.** Every number it prints is a floor, and the failure mode is reading a low one as "the CTA does not work" rather than "we could not see that it did".

## What This Is

**GuessSong** — a local party music guessing game built on **Next.js 15 App Router**. The host pastes a public Spotify playlist URL, adds player names, and plays short audio clips; everyone guesses out loud and the host awards points. **No login and no user accounts** — a single game's state lives in React state, handed off between pages via `sessionStorage`.

There *is* server-side storage, but it is deliberately narrow: a KV layer (`lib/kv.ts`) backed by Upstash Redis, used only for short-lived, TTL'd data — Mixed Playlist Mode's rooms (`lib/room.ts`) and IP rate limiting (`lib/rate-limit.ts`). Nothing is persisted per-user, and there are no tables or migrations. Local dev and tests fall back to an in-process `Map`, so neither needs a real Redis.

## Architecture

### Data Flow

1. **Setup** (`app/page.tsx`) — collects playlist URL, player names, clip duration (5–30s). On Start, calls `POST /api/playlist`, shuffles the returned tracks, writes the whole game payload to `sessionStorage` under the key `guesssong_game`, then navigates to `/game`.
2. **Game** (`app/game/page.tsx`, ~1200 lines, the heart of the app) — reads `guesssong_game` from sessionStorage on mount (redirects to `/` if absent) and runs a phase state machine:
   `waiting → playing → guessing → revealed → (next track | finished)`
3. **Audio previews** — Spotify deprecated `preview_url` (Nov 2024) and now returns `null` for *every* track on Client Credentials — measured 0/20 across four markets, which is why `Track` carries no `previewUrl` field at all. Every clip the game plays comes from iTunes or Deezer, resolved through `lib/preview-client.ts`. On mount it prefetches the whole game with one `POST /api/preview/batch`; anything that comes back unresolved falls back to `GET /api/preview` lazily, at the moment the host presses Play. Both search the **iTunes Search API** first and fall back to **Deezer**. Settled results are cached per track id in a ref (`previewCache`); tracks with no clip anywhere show a "no audio" state.

### API Routes (the only server code)

| Route | Purpose |
|---|---|
| `POST /api/playlist` | `{url}` → playlist name + tracks, via Spotify **Client Credentials** flow (`lib/spotify.ts`). Rejects Spotify editorial playlists (IDs starting `37i9` return 404 for new apps). |
| `GET /api/preview` | Track/artist/id → 30s preview URL (iTunes, then Deezer). No auth required. KV-cached by track id, including negative results. `&refresh=1` re-resolves a URL that stopped playing, on its own much tighter limit. |
| `POST /api/preview/batch` | `{tracks:[{id,name,artist}]}` → the same lookup for a whole game in one request. |
| `POST /api/room` | Creates a Mixed Playlist Mode room; returns room code, host token, expiry. |
| `POST /api/room/[code]/submit` | A player submits their playlist URL to the room. |
| `GET /api/room/[code]/status` | Poll for who has submitted so far. |
| `GET /api/room/[code]/pool` | Host consumes the room and gets the sampled, deduped track pool. |

**A room is a Redis hash, and its writers claim a field rather than rewriting the record.** `lib/room.ts` stores `meta`, `consumed` and one `p:<folded name>` per contributor under `room:v2:<CODE>`, with each contributor's tracks in their own `room:v2:<CODE>:t:<folded name>` key. Four rules hold it together, and each replaces something that was actively wrong:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Waynting/GuessSong](https://github.com/Waynting/GuessSong) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
