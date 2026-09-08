---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Jellyfin metadata plugin (.NET 8, GPL-3.0) that replaces the built-in TMDB
image provider with one that respects the library's language and matches
episode stills by title instead of by (Season, Episode) position. Targets
Jellyfin server ABI `10.10.0.0`. Pulls images via `TMDbLib`.

## Build / install / develop

```bash
dotnet build                       # compile
dotnet publish -c Release -o ./bin/publish   # what CI and build.sh do
./build.sh                         # publish + copy DLLs into local Jellyfin plugin dir, then restart Jellyfin
```

`build.sh` installs to `$HOME/Library/Application Support/jellyfin/plugins/LanguageAwareImages`
by default (macOS); override with `JELLYFIN_PLUGIN_DIR`. Only two assemblies
ship: `Jellyfin.Plugin.LanguageAwareImages.dll` and `TMDbLib.dll` (the Jellyfin
`Controller`/`Model` packages are reference-only — note `ExcludeAssets>runtime`
in the csproj — they're provided by the host server at runtime).

There are no automated tests. Verification is manual: build, install, restart
Jellyfin, and trigger a metadata refresh on a library item. Enable `Debug`
logging in Jellyfin to see this plugin's ranking/match decisions (it logs under
the `LanguageAwareImages` prefix).

## Release flow

Releases are driven entirely by git tags through `.github/workflows/build.yml`:

1. Bump `AssemblyVersion`/`FileVersion` in the `.csproj` and `version` in
   `meta.json` (and write the `changelog` in `meta.json`).
2. Push a tag `vX.Y.Z`. CI publishes, zips the two DLLs + `meta.json`, creates a
   GitHub Release (release notes = the tag commit's full message), and commits
   an updated `manifest.json` entry back to `main` with `[skip ci]`.

`manifest.json` is the Jellyfin plugin-repo feed users add; it is machine-edited
by CI — don't hand-edit version entries. The four-part version (`X.Y.Z.W`) and
`targetAbi` must stay consistent across the csproj, `meta.json`, and the
manifest.

## Architecture

Five `IRemoteImageProvider`s, all subclassing `LanguageAwareImageProviderBase`,
each gated to one Jellyfin entity type via `Supports`:

- `LanguageAwareMovieImageProvider` — Movie: Primary/Backdrop/Logo
- `LanguageAwareSeriesImageProvider` — Series: Primary/Backdrop/Logo
- `LanguageAwareSeasonImageProvider` — Season: Primary only (TMDB has no season backdrops/logos)
- `LanguageAwareEpisodeImageProvider` — Episode: Primary (still), title-matched
- `Plugin.cs` — registers config page; providers are auto-discovered by Jellyfin's DI.

Providers set `Order = -1` (vs. the built-in TMDB provider's `0`) so their
results are offered first. Jellyfin merges all providers' results and applies
its *own* downstream filter+sort, which is the source of most of the
non-obvious code (see below).

### The two key mechanisms

**Bucket ranking (`RankAndMap` in the base class)** — the core of the
poster/backdrop/logo logic. TMDB is asked (via `include_image_language`) for
preferred + optional original + fallback + optional textless (`null`) language
images in one call, then images are bucketed and ordered:

```
rank 0: preferred language
rank 1: fallback language   (default "en")
rank 2: original language   (opt-in, IncludeOriginalLanguage; dead-last by
                             default, or moved before fallback when
                             OriginalLanguageLast=false)
rank 3: textless (null)     (opt-in per image type; moved to rank -1, above the
                             preferred bucket, when PreferNoLanguageFor{Posters,
                             Backdrops,Logos} is set for that type)
excluded: dropped before mapping (no rank)
ORDER BY rank, then vote_count DESC tiebreak within a bucket
```

**Episode title-matching (`LanguageAwareEpisodeImageProvider`)** — fetches the
whole show's episodes once per `(showId, language)`, caches a normalised
`title → stillPath` map in a process-lifetime static `ConcurrentDictionary`,
and looks up the local episode's title against it. This decouples the image
from the (S,E) slot so alternative orderings (Disney+/DVD/chronological) get
the correct still. Always returns the title-matched still when found (not just
for out-of-sync episodes — see `meta.json` v0.7.4 changelog); returns empty on
no match so the built-in provider takes over with no regression.

### Two Jellyfin-pipeline workarounds (read before touching ranking)

Jellyfin re-processes the `RemoteImageInfo` list *after* the provider returns,
and that re-processing — not our own sort — is what actually decides the order
the user sees. Two facts about it (verified against `release-10.10.z`):

- **Filter** (`ProviderManager.GetImages`, when "show all languages" is off):
  keeps an image only if its `Language` is empty, equals the library metadata
  language (`item.GetPreferredMetadataLanguage()`, case-insensitive), or equals
  `"en"`. Nothing else survives — *not* a `PreferredLanguageOverride` value, not
  a regional variant like `pt-pt`, not the original language.
- **Sort** (`OrderByLanguageDescending` in `MediaBrowser.Model`): orders by a
  language score (library-language match = 4, empty = 3, `"en"` = 2, other = 0),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [superuser404notfound/jellyfin-plugin-language-aware-images](https://github.com/superuser404notfound/jellyfin-plugin-language-aware-images) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
