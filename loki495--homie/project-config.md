---
trigger: always_on
description: Self-hosted home lab dashboard. Laravel 13 + Livewire 4 (SFC) + Alpine.js + Tailwind v4 +
---

# Homie — project context

Self-hosted home lab dashboard. Laravel 13 + Livewire 4 (SFC) + Alpine.js + Tailwind v4 +
Flux UI (free tier). SQLite. Dockerized dev environment behind Traefik
(`homie.dev.local.test`).

## UI components

Sidebar manager forms/buttons (Groups, Cards, Discovery) use Flux UI components
(`<flux:input>`, `<flux:select>`, `<flux:textarea>`, `<flux:button>`) — free tier only,
no Pro license. Delete actions use `variant="ghost"` with a `!text-rose-*` class
override (not `variant="danger"`, which renders a filled red button — Flux's own
`text-zinc-800`/`dark:text-white` ghost-variant classes otherwise win the cascade tie
against a plain color override, so the `!` important modifier is required). The
off-canvas sidebar shell and the Groups/Cards/Discovery tab bar remain custom
Alpine — no Flux equivalent was worth the migration risk for either. Dark mode stays
on the project's own `Alpine.store('theme')` + inline FOUC-prevention script; Flux's
`@fluxAppearance`/`@fluxScripts` directives are included for the components' own
needs but the toggle button itself is not Flux's.
- The Cards sidebar list's filter input is plain Alpine (`x-model` + `x-show` per `<li>`
  matching against a `data-search` attribute) — no Livewire round-trip. This is the
  pattern for any future client-side-only filtering: cheaper than a Livewire property
  for something that never needs to touch the server.

## Card icons

`app/Support/DashboardIcons.php` searches the free homarr-labs/dashboard-icons index
(no API key) for icons matching common self-hosted app names, letting card creation
suggest an icon for recognized services. Icons are hotlinked from jsDelivr's CDN —
never downloaded or cached locally on this app's storage (a deliberate choice: keeps
things simple, no storage/cleanup concern, matches how Homarr/Dashy do it). The
`metadata.json` index itself is cached server-side for a day via `Cache::remember`.
`Card.icon` just stores a plain URL — either a resolved CDN link or a manually pasted
one, no distinction made at render time.

## Card API auth

`card_apis.auth_type` selects between `api_key` (sent as an `X-Api-Key` header — the
arr-stack convention) and `basic` (username/password, sent via `Http::withBasicAuth`).
Only one is active at a time based on `auth_type`; the unused fields are nulled out on
save so stale credentials from a previous auth-type choice don't linger. `password` is
encrypted at rest the same way `api_key` already was.

## Provider-specific API stats

`ApiProvider::fetcher()` maps every enum case to an `App\Support\ApiProviders\*Fetcher`
(implementing `ProviderFetcher`) — every provider in the enum has one, non-nullable, by
design (see "Adding a new provider" below). Each fetcher does its own HTTP calls and
returns `{status, summary, stats[], raw}` — `stats` is a small list of label/value pairs
rendered as chips on the card-api-widget instead of the generic "HTTP 200" line.
Endpoint shapes were verified against the gethomepage/homepage widget source (a mature
OSS project with working integrations for all of these) plus live calls against
Andres's own instances — not guessed. Notable per-provider quirks:
- Sonarr: `/api/v3/series` (count), `/api/v3/queue` and `/api/v3/wanted/missing` both
  paginated with a `totalRecords` field — request `pageSize=1` to avoid pulling the
  full list just for the count.
- Radarr v3 has no `/wanted/missing` endpoint (only existed in v1). Missing count is
  computed client-side from `/api/v3/movie`: monitored && !hasFile.
- NZBGet doesn't use an API key — it's HTTP Basic Auth (`ControlUsername`/
  `ControlPassword`), a JSON-RPC POST to `/jsonrpc` with `{"method": "status"}`. This is
  exactly what `auth_type = 'basic'` was built for.
- Prowlarr: same Servarr framework as Sonarr/Radarr, `X-Api-Key` header via
  `ApiHttpClient`. `/api/v1/indexer` (list, filter `enable === true` for the enabled
  count) and `/api/v1/indexerstats` (sum `numberOfGrabs`/`numberOfFailed*` across its
  `indexers` array) — no single endpoint gives an aggregate, so it fetches both.
- Bazarr does **not** follow the arr-stack header convention — it's `?apikey=` as a
  query string only (confirmed: an unauthenticated request to `/api/movies/wanted`
  returns a 401, and gethomepage/homepage's working integration only ever sends the key
  in the query string). `BazarrFetcher` calls `Http::get()` directly instead of going
  through `ApiHttpClient`, since that helper's basic-auth/header logic doesn't apply
  here. Missing-subtitle counts come from `/api/movies/wanted` and
  `/api/episodes/wanted`, both `{"total": N, ...}`.

Adding a new provider: add the enum case, a Fetcher implementing `ProviderFetcher`, and
one `match` arm in `ApiProvider::fetcher()`, all in the same change — the widget needs
no changes. Every case must resolve to a real fetcher (the return type is
non-nullable); don't add an enum case before its fetcher exists.

## Discovery: host-network containers need an inspect fallback

`docker ps`/`/containers/json` report an empty `Ports` for containers running with
`--network host` — there's no mapping to report, the container's ports *are* the host's
ports directly. Without a fallback, every host-network container with no Traefik label

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [loki495/homie](https://github.com/loki495/homie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
