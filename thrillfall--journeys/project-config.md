---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Nextcloud app (`journeys`, app id `journeys`, namespace `OCA\Journeys`) that clusters a user's photos by time + location into "journey" albums, and optionally renders each album to an MP4 via `ffmpeg`.

Hard dependencies on other Nextcloud apps:
- **Memories** — this app does not scan the filesystem. It reads photo rows (id, path, datetaken, lat, lon) from the Memories index table (`oc_memories`), so the set of clusterable photos is whatever Memories has indexed.
- **Photos** — albums are created through `OCA\Photos\Album\AlbumMapper`. Only `fileid` is used to add photos (mount-agnostic), not paths.

Supported Nextcloud: `30–33`. PHP `>=8.0`.

## Commands

Frontend build (webpack, single Vue 2 bundle → `js/journeys-personalSettings.js`):
```sh
npm ci                    # first time
npx webpack --mode production     # production bundle (same as `npm run build`)
npx webpack --mode development --watch   # dev watch
```
`js/` is gitignored and must be rebuilt before testing the personal settings UI. The Makefile targets `build-js`, `watch-js`, `test`, `lint` reference npm scripts (`dev`, `watch`, `test:unit`, `lint`) that are **not defined** in `package.json` — only `build` exists. Don't trust those Makefile targets for day-to-day work; invoke webpack / phpunit directly.

PHP tests (PHPUnit, bootstrapped via Composer autoload — no Nextcloud server required for the `tests/Service/*` suite, which uses pure unit tests):
```sh
vendor/bin/phpunit                                    # all tests
vendor/bin/phpunit tests/Service/ClustererTest.php    # single file
vendor/bin/phpunit --filter testClusterImagesWith...  # single test
```

Packaging for the Nextcloud appstore:
```sh
make appstore     # produces build/artifacts/journeys.tar.gz
```

OCC commands (run from the Nextcloud root as the web user). Primary ones:
```sh
php occ journeys:cluster-create-albums <user> [--from-scratch] [--include-group-folders] \
    [--include-shared-images] [--no-merge] [--debug-splits] [--from <date>] [--to <date>] [--last-years N]
php occ journeys:list-clusters <user>
php occ journeys:remove-all-albums <user>           # only removes albums tracked by this app
php occ journeys:render-cluster-video <user> <albumId>
php occ journeys:render-cluster-video-landscape <user> <albumId>
php occ journeys:show-latest-cluster-end <user>
```

## Architecture

### Dependency injection — explicit, not auto-wired

Services are registered in `appinfo/services.xml` (Symfony DI container XML). **Auto-wiring is not used for this app's own services** — if you add a new service with constructor dependencies on other services in `OCA\Journeys\Service\*`, you must add a `<service>` entry in `services.xml` listing each `<argument>`. A silent failure mode we've hit before: forgetting to register a service inside `<services>` → the daily cron appears to work but the dependent feature doesn't run (see 0.20.1 changelog).

`lib/AppInfo/Application.php` is the `IBootstrap` entry point — it only registers the notifier. App config, routes, commands, background jobs, and settings are declared in `appinfo/info.xml` and `appinfo/routes.php`.

### Clustering pipeline

`ClusteringManager::clusterForUser()` is the orchestrator for both OCC and the daily cron. Flow:

1. `ImageFetcher::fetchImagesForUser()` queries `oc_memories` joined against `oc_filecache` / `oc_mounts` to return `Image[]`. Three sources are blended based on flags: home storage (always), Group Folders / external mounts (`includeGroupFolders`), incoming user shares scoped to their mount root (`includeSharedImages`). Per-source counts are exposed via `getLastFetchStats()` / `getLastFileSources()`.
2. `HomeService` / `HomeLocationDetector` determine the user's home coordinate (manual override via user config, or auto-detected).
3. Timeline is segmented into near-home vs away-from-home blocks (home-aware mode is default). Each segment is clustered independently by `Clusterer::clusterImages()` using per-segment time + distance thresholds. Distance checks anchor to the **last geolocated image** in the current cluster (`prevGeo`), so a run of no-GPS photos cannot bridge a large spatial jump.
4. **`ClusterMerger::mergeAdjacent()` post-processes the cluster list**, stitching adjacent clusters that are in the same country (resolved via `ClusterLocationResolver::resolveClusterCountry()`, OSM `admin_level=2`) and within 7 days of each other. This fixes over-splitting of multi-city road trips and long vacations with photo-less rest days — cases where threshold tuning alone can't help (the user tested `awayDistanceKm=200` in production and multi-city still split). Only away-from-home clusters are merged when home is known; near-home clusters never merge. Disable with `--no-merge` CLI flag or the `mergeAdjacent` user setting.
5. `ClusterLocationResolver` + `SimplePlaceResolver` resolve a place name for each cluster (uses Memories' Places tables; falls back automatically when PostGIS functions are unavailable).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thrillfall/journeys](https://github.com/thrillfall/journeys) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
