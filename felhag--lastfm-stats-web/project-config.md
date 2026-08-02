---
trigger: always_on
description: Guidance for AI agents working in this repository. Humans, see `README.md`.
---

# AGENTS.md

Guidance for AI agents working in this repository. Humans, see `README.md`.

## What this project is

An Angular app that renders extra statistics on top of a user's listening history. There are two deployable apps that share the bulk of the code:

- **lastfm-stats** — live at https://lastfmstats.com, sources data from the last.fm API.
- **spotify-stats** — deployed to https://spotifystats.app. Built from this repo; the `felhag/spotify-stats-web` repo exists only to host the `gh-pages` branch and custom domain (GitHub Pages is one custom domain per repo). Its `master` branch is unused.

Keep the two apps as similar as possible. Real differences (so far): the noun used in the UI (`scrobbles` vs `plays`) and how data is imported (last.fm paginates over its API; Spotify loads everything at once from a user-supplied zip).

## Repo layout

```
projects/
  shared/         Angular library — most code lives here
    src/lib/
      app/        AppComponent, root model.ts (Scrobble, TempStats, Artist, …)
      service/    StatsBuilderService, ScrobbleStore, DatabaseService, ExportService, SettingsService, …
      charts/     Highcharts wrappers (race chart, timeline, scatter, wordcloud, …)
      lists/      Per-tab list components (artist/album/track/scrobble)
      dataset/    Dataset tab + modal
      stats/      Stats shell component (tabs)
      general/    General overview tab
      enrichment/ MusicBrainz / last.fm artist info enrichment
      …
  lastfm-stats/   Thin wrapper: home page, ScrobbleRetrieverService (last.fm API), URL service
  spotify-stats/  Thin wrapper: home page (zip parsing), SpotifyItemService, URL service
```

Each app is bootstrapped in `projects/<app>/src/main.ts`. The wiring there is what defines the differences between the apps — see "App differences" below.

## The `TempStats` model

Everything downstream of data loading is fed by a single `TempStats` object (`projects/shared/src/lib/app/model.ts`). Scrobbles are streamed into `StatsBuilderService` (`projects/shared/src/lib/service/stats-builder.service.ts`), which walks them once and accumulates streaks, per-month buckets, hour/day/year histograms, seen artists/albums/tracks, milestones, etc.

**Performance matters here.** A large profile can be hundreds of thousands of scrobbles. The builder is hot path:

- Avoid allocating per scrobble. Mutate the accumulator, don't rebuild it.
- Don't introduce per-scrobble RxJS pipelines, deep clones, or `Object.entries`/`Object.values` traversals over `seenArtists`-sized maps inside the loop.
- The builder runs in two modes (see `tempStats` setup): incremental (`autoUpdate` on — `scan` over chunks, accumulator reused) and full rebuild (settings changed, or load completed with `autoUpdate` off — starts from `emptyStats()`). Changes must keep both modes correct.
- Downstream lists/charts subscribe to `tempStats` and derive their own views; don't push view-specific data into `TempStats` if it can be computed downstream cheaply.

If you're tempted to add a field to `TempStats`, first check whether the same value can be computed lazily by a single list or chart component. Adding to `TempStats` costs every user on every rebuild.

## App differences and the `translate` pipe

The user-facing noun (`scrobble` vs `play`) is **never hardcoded** in shared code. Use the `translate` pipe / `TranslatePipe` (`projects/shared/src/lib/service/translate.pipe.ts`). Keys today: `scrobble` / `scrobbles` / `scrobbled` — bound to `play` / `plays` / `played` in spotify-stats. Bindings are set up in each app's `main.ts` via `Shared.translationsProvider(...)`.

When adding new shared UI strings, decide:

- Same word in both apps → hardcode it.
- Different word → add a translation key, wire it in both `main.ts` files, and use the pipe.

The scrobbles tab is also routed under a different path in each app: `scrobbles` for last.fm, `plays` for spotify. This is passed into `Shared.routerProvider(...)` from `main.ts`.

Other DI knobs set per app in `main.ts`:

- `AbstractItemRetriever` — `ScrobbleRetrieverService` (paginates the last.fm API) vs `SpotifyItemService` (hands off pre-imported scrobbles from the home page's zip parser).
- `AbstractUrlService` — builds external links to artist/album/track pages on last.fm or Spotify.
- `App` enum token — used by `DatabaseService` to pick the IndexedDB name (`lastfmstats` vs `spotifystats`) and by anything else that needs to branch on platform.

If you need to branch on app, prefer injecting `App` or one of these abstract services over duplicating components.

## Data import / export (mostly last.fm)

Last.fm's API is slow for large profiles, so caching the data client-side is a first-class feature:

- `ExportService` (`projects/shared/src/lib/service/export-service.ts`) — JSON and CSV export.
- `DatabaseService` (`projects/shared/src/lib/service/database.service.ts`) — IndexedDB via Dexie. Scrobbles plus `ArtistInfo` enrichment are persisted per user; on revisit they load instantly and only the delta is fetched from the API.
- `ScrobbleImporter` (`projects/shared/src/lib/service/scrobble-importer.service.ts`) — bridges the home page's parsed scrobbles into the stats flow.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [felhag/lastfm-stats-web](https://github.com/felhag/lastfm-stats-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
