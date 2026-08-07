---
trigger: always_on
description: This repository contains a Navidrome lyrics plugin that scrapes lyrics from Musixmatch. Navidrome loads the plugin as a WASI WebAssembly module and calls its lyrics extension point. The plugin searches Musixmatch for a requested artist/title, fetches the matched lyrics page, parses embedded Next.js data, and returns synced LRC lyrics when available or plain lyrics as a fallback.
---

# AGENTS.md

## Purpose

This repository contains a Navidrome lyrics plugin that scrapes lyrics from Musixmatch. Navidrome loads the plugin as a WASI WebAssembly module and calls its lyrics extension point. The plugin searches Musixmatch for a requested artist/title, fetches the matched lyrics page, parses embedded Next.js data, and returns synced LRC lyrics when available or plain lyrics as a fallback.

Treat this file as the first source of project context. Search the codebase only when this file is incomplete, stale, or contradicted by the current task.

## Repository Layout

- `plugin/` is the Go module and plugin source root.
- `plugin/main.go` registers the Navidrome lyrics plugin and exports `nd_on_init`.
- `plugin/lyrics.go` implements Navidrome's `GetLyrics` method and delegates to `plugin/musixmatch`.
- `plugin/musixmatch/` contains Musixmatch-specific desktop API, website search/fetch fallback, normalization, and LRC conversion logic.
- `plugin/utils/` contains shared config, logging, constants, and PDK HTTP helpers.
- `plugin/manifest.json` defines Navidrome plugin metadata, HTTP permissions for `apic-desktop.musixmatch.com` and `www.musixmatch.com`, cache permission for the desktop API token, and plugin config schema.
- `just/` and `justfile` define local build, test, install, and cleanup commands.
- `flake.nix` defines the Nix dev shell and reproducible TinyGo/Nix package build.

- Ignore anything inside `navidrome-instance`

## Runtime Flow

1. `plugin/main.go` calls `lyrics.Register(&plugin{})` in `init`.
2. Navidrome calls `(*plugin).GetLyrics` in `plugin/lyrics.go`.
3. `GetLyrics` calls `musixmatch.FetchLyrics` and prefixes returned errors with `navidrome-musixmatch-plugin: `.
4. `musixmatch.FetchLyrics` logs the requested track and first tries the free, unofficial Musixmatch desktop API.
5. `fetchLyricsFromDesktopAPI` gets an anonymous desktop `user_token` via `token.get`, caches it with Navidrome's host cache for 10 minutes, then calls `macro.subtitles.get`.
6. Desktop API lyrics are returned in this order: richsync converted to LRC, subtitle LRC, then plain lyrics.
7. If the desktop API fails or returns no lyrics, `FetchLyrics` falls back to website scraping.
8. `searchForTrack` normalizes artist/title, fetches the Musixmatch website search page, extracts `<script id="__NEXT_DATA__">`, unmarshals embedded search JSON, then chooses `bestMatch` when it is a `track`, otherwise `tracks[0]`.
9. `scrapeWebsiteLyricsForTrack` fetches the Musixmatch lyrics page for the selected `commontrack_vanity_id`, extracts `<script id="__NEXT_DATA__">`, and unmarshals the JSON payload.
10. Website fallback lyrics are returned in this order: synced LRC from `trackStructureList`, synced LRC from `subtitle`, then plain `lyrics.body`.

## Key Files

- `plugin/musixmatch/1_desktop.go` implements the unofficial desktop API path at `apic-desktop.musixmatch.com/ws/1.1`, including 10-minute token caching and `macro.subtitles.get` parsing.
- `plugin/musixmatch/2_search.go` uses `MusixmatchSearchPageURL`, extracts the search page's `__NEXT_DATA__`, and parses `pageProps.data.openSearch.data.opensearchTrackSearch.body`.
- `plugin/musixmatch/3_website_scraper.go` uses `nextDataRe` to parse the Musixmatch page and reads `props.pageProps.data.trackInfo.data`.
- `plugin/musixmatch/4_website_scraper__lyrics_parser.go` converts Musixmatch timestamp totals into LRC tags like `[mm:ss.hh]`.
- `plugin/musixmatch/9_utils.go` normalizes input by lowercasing, stripping diacritics, removing bracketed text, removing common dash suffixes, and collapsing whitespace.
- `plugin/utils/http.go` sends GET requests with PDK HTTP, `Accept-Language: en`, configured `Accept`, configured `User-Agent`, and optional Musixmatch cookies.
- `plugin/utils/constants.go` contains the hardcoded Musixmatch URLs and default headers.

## Configuration

Config keys come from `plugin/manifest.json` and `plugin/utils/constants.go`.

- `musixmatch_user_token` is sent as the `musixmatchUserToken` cookie when set.
- `musixmatch_captcha_id` is sent as the `captcha_id` cookie only when `musixmatch_user_token` is also set.
- `musixmatch_user_agent` defaults to mobile Safari.
- `musixmatch_http_accept` defaults to a browser-like HTML accept header.

The desktop API path does not require user-entered Musixmatch cookies or an official API key. It automatically fetches an anonymous desktop `user_token` and caches it with Navidrome's `host.Cache*` service for 10 minutes. The manifest must keep the `cache` permission so Navidrome exports the required cache host functions.

Note: `musixmatch_captcha_id` exists in the manifest schema, but it is not currently included in `uiSchema.elements`, so verify the Navidrome plugin settings UI before assuming users can edit it there.

## Build And Validation

Run commands from the repository root unless a command states otherwise.

- Enter the Nix dev shell when available: `nix develop`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Myzel394/navidrome-musixmatch-plugin](https://github.com/Myzel394/navidrome-musixmatch-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
