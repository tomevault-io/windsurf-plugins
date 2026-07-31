---
trigger: always_on
description: Stream Spigot is a collection of tools for consuming real time-ish data sources at a manageable pace. The active application is the Cloudflare/SvelteKit worker in `worker/`. The old App Engine implementation and retired tools live in `legacy/` for reference only.
---

# Stream Spigot Agent Notes

## Project Overview

Stream Spigot is a collection of tools for consuming real time-ish data sources at a manageable pace. The active application is the Cloudflare/SvelteKit worker in `worker/`. The old App Engine implementation and retired tools live in `legacy/` for reference only.

Masto Feeder signs in with Mastodon, reads the user's timeline, and renders it as an Atom feed or debug HTML. Sky Feeder signs in with Bluesky through AT Protocol OAuth, reads the user's home timeline, and renders it as Atom, JSON Feed, or debug HTML. Tweeter Feeder reads public Twitter/X timelines through private web endpoints and renders them through the same shared feed/status pipeline.

## Active Architecture

- SvelteKit routes live under `worker/src/routes`.
- Masto Feeder request handling is centralized in `worker/src/lib/masto-feeder/controller.ts`.
- The controller owns auth/session/KV concerns and delegates timeline feed rendering to `worker/src/lib/masto-feeder/feed.ts`.
- `worker/src/lib/masto-feeder/feed.ts` fetches Mastodon API data and adapts statuses into provider-neutral status objects.
- Tweeter Feeder request handling is centralized in `worker/src/lib/tweeter-feeder/controller.ts`.
- `worker/src/lib/tweeter-feeder/fetcher.ts` handles Twitter/X auth, private GraphQL fetches, response parsing, session cooldowns, and caching.
- `worker/src/lib/tweeter-feeder/status-adapter.ts` converts parsed Twitter/X tweets into provider-neutral `Status` objects.
- Sky Feeder request handling is centralized in `worker/src/lib/sky-feeder/controller.ts`.
- `worker/src/lib/sky-feeder/oauth.ts` builds the AT Protocol OAuth client, exposes client metadata/JWKS data, and restores OAuth sessions.
- `worker/src/lib/sky-feeder/feed.ts` fetches the Bluesky home timeline with `@atproto/api`.
- `worker/src/lib/sky-feeder/status-adapter.ts` converts Bluesky feed posts, reposts, quotes, images, videos, and external cards into provider-neutral `Status` objects.
- Small shared feeder helpers live in `worker/src/lib/feeder`.
- `worker/src/lib/status/feed.ts` renders normalized statuses as Atom or debug HTML with `svelte/server`.
- Shared feed-reader-friendly status UI lives in `worker/src/lib/components`, centered on `StatusDisplay.svelte` and related `StatusDisplay*` components.
- Provider-neutral status types live in `worker/src/lib/status`.

## Tweeter Feeder Twitter/X Data Loading

Tweeter Feeder does not use the official Twitter/X developer API. It follows the
same general approach as [Nitter](https://github.com/zedeus/nitter): make
authenticated requests to Twitter/X's private web GraphQL endpoints, then adapt
the returned web-client JSON into a stable local shape. Use Nitter as the
reference implementation for provider quirks, endpoint behavior, and timeline
parsing details, but keep the Stream Spigot implementation small and focused on
feed generation.

Runtime sessions are stored in KV under `tweeter-feeder:sessions`. Each session
is a cookie-derived credential bundle with `auth_token` and `ct0`. The fetcher
builds browser-like request headers and sends a matching `ct0` cookie and
`x-csrf-token` header to the private GraphQL API.

The high-level fetch flow is:

1. Normalize requested usernames and choose an available session deterministically.
2. Resolve each screen name to a Twitter/X user id with `UserResultByScreenNameQuery`.
3. Fetch that user's timeline with `UserTweets`.
4. Parse users, tweets, reposts, quotes, cards, polls, media, entities, and profile images out of the GraphQL response.
5. Cache parsed user and timeline results in KV, and temporarily cool down sessions that hit auth or rate-limit style failures.
6. Convert parsed Twitter/X tweets to the shared `Status` shape before rendering with the common Atom/debug HTML renderer.

Twitter/X private endpoints and response shapes change without notice. When auth
or parsing breaks, compare the current web-client request with `fetcher.ts`,
check that `ct0` is complete and matches in both cookie and header, and consult
the Nitter repo for the closest known handling of current Twitter/X behavior.

## Sky Feeder Bluesky Data Loading

Sky Feeder uses AT Protocol OAuth rather than app passwords. It depends on
`@atproto/api` and `@atproto/oauth-client-node`; keep `nodejs_compat` enabled
in both Wrangler configs. The deployed Worker also needs the
`ATPROTO_OAUTH_PRIVATE_JWK` secret. Generate it from `worker/` with
`npm run gen:atproto-key`, then store the generated JSON object with
`wrangler secret put ATPROTO_OAUTH_PRIVATE_JWK`.

AT Protocol discoverable OAuth client IDs must be HTTPS URLs that are not
loopback hosts and can be fetched by the Bluesky/ATProto OAuth server. Local
HTTP is enough to smoke-test `/sky-feeder`,
`/sky-feeder/oauth-client-metadata.json`, and `/sky-feeder/jwks.json`, but a
complete Bluesky sign-in requires the deployed HTTPS Worker URL or a public
HTTPS tunnel such as Tailscale Funnel pointed at the local Vite dev server.

Sky Feeder data is stored in KV under `sky-feeder:` keys. OAuth state entries

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mihaip/streamspigot](https://github.com/mihaip/streamspigot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
