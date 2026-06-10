---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- `npm run build`: `tsc` to `dist/` (ESM, NodeNext). `tests/`, `references/`, `dist/` are excluded.
- `npm test`: `vitest` in watch mode.
- `npm run test:run`: one-shot run of the whole suite (unit + live E2E, ~60s total).
- `npx vitest run tests/e2e`: just the live providers.
- `npx vitest run tests/e2e/allmanga.test.ts`: single E2E file.
- `npx vitest run -t "rewriteManifest"`: single test by name pattern.
  `pnpm` is also configured (`pnpm-lock.yaml`, `pnpm-workspace.yaml`): either pkg manager works.

Requires Node 20+ and `ffmpeg` on `PATH` (E2E suite shells out to it).

## Architecture

The SDK has three layers, all wired around a single `HttpClient`:

**1. Transport (`src/transport/`)**: site-agnostic plumbing.

- `HttpClient` wraps `fetch` with a **curl fallback** that fires automatically on Node when `fetch` errors out (timeout, TLS quirks, anti-bot rejection). The fallback synthesizes a `Response`-shaped object so callers don't see the difference. It also supports two proxy routing modes (`prepend` puts the proxy in front of `host/path`; `query` passes the URL as a query param): `requestUrl(url)` is the single chokepoint for that rewrite.
- `DomRegistry` is a global single-parser registry. `BrowserDomParser` works in browsers; in Node, consumers (and the E2E tests' `beforeAll`) must shim `globalThis.DOMParser` via `linkedom` before any provider parses HTML. Providers call `DomRegistry.parse(html)`: they never touch `DOMParser` directly.
- `HlsUtils.rewriteManifest` rewrites every URI line in an `.m3u8` (including `URI="…"` inside `#EXT-X-KEY` / `#EXT-X-MAP`) so chunk fetches go through the same proxy as the manifest fetch.

**2. Extractors (`src/extractors/`)**: stateless, take only an embed URL and an `HttpClient`, return `IVideoPayload[]` (empty if they can't recover a direct stream). `BaseExtractor` is the contract. They're independently usable: a consumer can hand any embed URL to `BloggerExtractor` without involving a provider.

**3. Providers (`src/providers/`)**: site-specific. `BaseProvider` defines `search` → `fetchContentUnits(mediaId)` → `resolveStream(unitId, language?)`. `fetchContentUnits` is **language-agnostic** and returns one unified list; each `IContentUnit` carries `availableLanguages: ContentLanguage[]` so the caller picks the translation at `resolveStream` time. Providers may optionally implement `fetchUnitTracks(unitId, language?): Promise<IUnitTracks>` to expose subtitle/quality metadata without paying the `resolveStream` cost. `IVideoPayload.subtitles?: ISubtitleTrack[]` carries playable VTT URLs alongside the stream. Each provider composes one or more extractors:

- `AnimeParadiseProvider`: `api.animeparadise.moe` REST. `/anime/{id}/episode` for the list (sub only). `/ep/{uid}?origin={animeId}` returns the playable HLS link **and** `subData`, which `normalizeSubtitleEntries` (in `utils/subtitles.ts`) turns into VTT-only `ISubtitleTrack[]`. Implements `fetchUnitTracks` cheaply (just `/ep`, no stream URL resolution).
- `AllmangaProvider`: AllAnime GraphQL → AES-CTR-decrypted `tobeparsed` payload → `Mp4UploadExtractor`, with a `clock.json` fallback for wixmp/sharepoint sources. Source URLs are obfuscated with a `--<hex>` scheme XOR'd with `0x38`; see `decodeAllAnimeSource`. `fetchContentUnits` merges `availableEpisodesDetail.sub` + `.dub` + `.raw` into a single language-agnostic list; unit IDs are `${mediaId}/${epStr}` (legacy `${mediaId}/${epStr}/${lang}` IDs still resolve).
- `AnikotoProvider`: HTML scrape of `anikototv.to`; uses `anikotoapi.site` for episodes, then delegates to MegaPlay embed for stream/subtitles.
- `GogoanimeProvider`: HTML scrape of `anineko.to`; vibeplayer embed → `master.m3u8` via `GenericHlsExtractor`.
- `GoyabuProvider`: pulls a Blogger token from `playersData`, calls Google `batchexecute` to recover the `googlevideo.com` URL via `BloggerExtractor`.
- `MangadexProvider`: Official JSON API at `api.mangadex.org` for high-quality manga.
- `MangapillProvider`: HTML scrape of `mangapill.com` for manga.
- `MegaPlayProvider`: AniList GraphQL for search/episodes; resolves directly against MegaPlay's mapping endpoints.
- `WeebcentralProvider`: HTML scrape of `weebcentral.com` for manga.

All public surface is re-exported from `src/index.ts`, including the shared subtitle utilities (`normalizeSubtitleEntries`, `proxifySubtitleUrl`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hexxt-git/anime-sdk](https://github.com/hexxt-git/anime-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
