---
trigger: always_on
description: Cloudflare Worker rendering a Spotify "recently played" SVG card for GitHub profile READMEs, plus a static configurator. Replaces a Next.js/Vercel app; same Realtime Database and schema, so existing authorizations carry over.
---

# spotify-recently-played

Cloudflare Worker rendering a Spotify "recently played" SVG card for GitHub profile READMEs, plus a static configurator. Replaces a Next.js/Vercel app; same Realtime Database and schema, so existing authorizations carry over.

## Commands

```bash
npm run dev          # wrangler dev on :8787
npm run typecheck
npm test
npm run deploy
```

Pushing to `main` runs typecheck + tests in GitHub Actions; Cloudflare Workers Builds deploys.

## Conventions

American spellings throughout - code, comments, docs and UI. The URL parameters are `bg_color`, `text_color` and so on, and prose that says "color" beside them reads as a different thing.

## What the rendering context forces

GitHub renders the card inside an `<img>`, proxied by Camo:

- **The SVG must be self-contained.** No external fonts, images, CSS or JS loads. Cover art is fetched server-side and inlined as a base64 data URI.
- **The card has a border, and it isn't optional.** Removed once, then put back: a borderless card floats ambiguously, and every theme already carries a `border` measured against its palette. A toggle would only add a parameter nobody needs.
- **Links, tooltips and `:hover` are inert inside an `<img>`**, so nothing may depend on them, but they work when opened directly. Track title, cover art and logo are links; title and timestamp carry `<title>` tooltips, which must be the **first child** of the element - why `tooltip()` wraps rather than appends.
- **The logo carries a transparent hit rect**, since a wordmark is mostly gaps between letters.
- **CSS animation does run** (progress bar); SMIL and external stylesheets aren't needed.
- **Errors return HTTP 200 with a valid SVG.** A 4xx renders as a broken-image icon and poisons Camo's cache.
- **Everything must finish inside Camo's ~10s socket timeout.** Firebase read, token refresh, one to three Spotify calls, then art - all share one `Deadline` (`src/util/deadline.ts`). Don't add independent timeouts, they sum.

## Spotify

### Read the OpenAPI schema, not the docs pages

`https://developer.spotify.com/reference/web-api/open-api-schema.yaml`. Several fields carry `deprecated: true`:

| Field | Would have been |
| --- | --- |
| `TrackObject.popularity`, `ArtistObject.popularity` | a popularity bar |
| `ArtistObject.genres` | genre tags |
| `ArtistObject.followers`, `PrivateUserObject.followers` | a stats strip like the Last.fm card's |
| `TrackObject.preview_url` | 30-second previews |

No `stats` option here: every number Spotify exposes about an account is deprecated.

### Things the schema does not tell you

- **`/me/player/currently-playing` returns 204 with an empty body** when nothing is playing. The schema documents 200/401/403/429 only, so `response.json()` throws on the most common case.
- **An insufficient scope answers 401, not 403.** Verified against a live token holding only `user-read-recently-played`: `/me/player/currently-playing` returns **401 "Permissions missing"** while `/me/player/recently-played` returns 200. The schema documents 403 for a bad OAuth request, and building around that broke every existing user's card. `optional()` in `src/index.ts` swallows 401 deliberately - that is not a bug.
- **`/me` works with any valid token.** `user-read-private` and `user-read-email` gate `country`, `email`, `product` and `explicit_content` (all deprecated). `id`, `display_name` and `images` always return.
- **`ContextObject` has no name** (only type/href/uri), so "played from Discover Weekly" costs an extra request per row.
- **`currently_playing_type` must be checked.** Episodes, ads and unknown items all reach that endpoint.

### Scopes and the migration

New authorizations request `user-read-recently-played` and `user-read-currently-playing`. Every account that authorized the Vercel app granted only the first, so **the now-playing row 401s for all of them until they reconnect** - the card drops that section and renders the rest. Never fail the card over an optional section.

`user-top-read` is deliberately not requested: the feature it would serve isn't built, and asking for a scope ahead of the feature is how consent screens get alarming.

### Terms

Spotify's Developer Terms require attribution, current data and no indefinite storage. So: the logo/title/aria-label always name Spotify regardless of `logo`; track data is cached 20-60s; nothing is aggregated or fed to a model.

Cover art is cached 24h. URLs are content-addressed (`i.scdn.co/image/<hash>`), so a new cover is a new URL - this caches an immutable asset, not stale metadata. The Vercel proxy sent `max-age=86400, immutable`.

## Tokens and Firebase

`firebase-admin` doesn't run on Workers, so `src/firebase.ts` signs a service-account JWT with WebCrypto and calls the Realtime Database REST API directly. Two npm alternatives exist; both last published 2023 - not worth adding to the credential path for one function.

- **Both Google scopes are required.** `firebase.database` alone fails - the read looks like a missing node rather than an auth error. `userinfo.email` must be there too.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JeffreyCA/spotify-recently-played-readme](https://github.com/JeffreyCA/spotify-recently-played-readme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
