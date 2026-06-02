---
trigger: always_on
description: > Single source of truth for project architecture, commands, conventions, and contracts.
---

# AGENTS.md - PawTunes Implementation Reference

> Single source of truth for project architecture, commands, conventions, and contracts.

## Project

PawTunes — web-based internet radio player. PHP 7.4+ backend (no framework), TypeScript frontend compiled to ES2020 ESM via esbuild.

- **Version**: 1.0.7
- **License**: MPL-2.0
- **Repository**: https://github.com/Jackysi/PawTunes

## Tech Stack

| Layer         | Technology                                       |
| ------------- | ------------------------------------------------ |
| Backend       | PHP 7.4+ (vanilla, no framework)                 |
| Frontend      | TypeScript -> ES2020 ESM via esbuild             |
| Styles        | SCSS -> CSS via Dart Sass 1.78.0                 |
| Build         | Gulp 5 (parallel: SCSS, TypeScript, JS concat)   |
| Panel views   | BladeOne (Blade template engine)                 |
| Player views  | `{{$var}}` interpolation (`Helpers::template()`) |
| Visualization | audioMotion-analyzer ^4.5.0                      |
| Testing       | PHPUnit 10                                       |

## Essential Commands

```bash
# Install dependencies
npm install
composer install

# Development build + watch
npm run dev

# Production build (SCSS + TypeScript + JS minification)
npm run build

# Create release ZIP
npm run release

# Run all tests (requires a running local server at PAWTUNES_BASE_URL)
vendor/bin/phpunit

# Run a single test file
vendor/bin/phpunit tests/Unit/PawTunesTest.php

# Run a specific test suite
vendor/bin/phpunit --testsuite Feature
vendor/bin/phpunit --testsuite Unit
```

## Source Structure

### Core Backend (`inc/`)

```
inc/
├── autoload.php              # SPL autoloader: namespace\Class -> inc/namespace/Class.php
├── handler.php               # Track info request handler (included by index.php)
├── handle-artwork.php         # Artwork proxy/resize endpoint (included by index.php)
├── playlist-handler.php       # Dynamic M3U/PLS/ASX generation (included by index.php)
├── config/
│   ├── general.php            # Runtime settings (gitignored, writable by panel)
│   ├── general.example.php    # Config template for fresh installs
│   └── channels.php           # Channel definitions (gitignored, writable by panel)
└── lib/
    ├── Helpers.php            # Abstract base: HTTP client, template engine, file utils
    ├── PawTunes.php           # Core: config, channels, cache init, artwork orchestration
    ├── Cache.php              # Multi-backend cache (disk/apcu/redis/memcached)
    ├── PawException.php       # RuntimeException subclass for recoverable errors
    ├── ImageResize.php        # GD/Imagick image crop and resize
    ├── bundle.crt             # CA certificate bundle for CURLOPT_CAINFO
    └── PawTunes/
        ├── StreamInfo/        # Track info provider plugins
        │   ├── TrackInfoInterface.php
        │   ├── TrackInfo.php          # Abstract base with shared helpers
        │   ├── Shoutcast.php          # Shoutcast admin XML API
        │   ├── ShoutcastPublic.php    # Shoutcast public JSON feed
        │   ├── Icecast.php            # Icecast admin XML API (auth required)
        │   ├── IcecastPublic.php      # Icecast public JSON endpoint
        │   ├── Azuracast.php          # AzuraCast WebSocket + REST API
        │   ├── Centovacast.php        # CentovaCast widget API
        │   ├── SAM.php                # SAM Broadcaster database query
        │   ├── Direct.php             # Raw ICY-METADATA stream reading
        │   └── Custom.php             # User-defined HTTP endpoint
        └── Artwork/           # Artwork provider plugins
            ├── Artwork.php            # Abstract base: resolution pipeline + download
            ├── iTunes.php             # iTunes Search API (no key required)
            ├── Spotify.php            # Spotify Web API (client_id:client_secret)
            ├── LastFM.php             # LastFM XML API
            ├── FanArtTV.php           # FanArt.TV API
            └── Custom.php             # URL template with {{$artist}}/{{$title}}
```

### Frontend Source (`src/`)

```
src/
├── player/ts/
│   ├── pawtunes.ts            # Main player class (extends HTML5Audio)
│   ├── html5-audio.ts         # HTML5 Audio API wrapper
│   ├── html5-audio-mse.ts     # MediaSource Extension support
│   ├── pawtunes-events.ts     # Event bus
│   ├── pawtunes-ws.ts         # WebSocket handler for live updates
│   ├── storage.ts             # LocalStorage wrapper with expiration
│   ├── types.ts               # Global type definitions
│   └── types/
│       ├── player.ts          # Channel, OnAir, PawMediaSource interfaces
│       └── translation.ts     # i18n type definitions
├── panel/scss/
│   └── style.scss             # Panel stylesheet entry (Prahec UI Framework)
└── templates/
    ├── pawtunes-tpl.ts        # PawTunes template script
    └── simple.ts              # Simple template script
```

### Control Panel (`panel/`)

```
panel/
├── index.php                  # Entry point: auth gate, routing
├── login.php                  # Login form + POST handler
├── home.php                   # Dashboard
├── channels.php               # Channel list
├── channels.edit.php          # Channel create/edit form + handler

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jackysi/PawTunes](https://github.com/Jackysi/PawTunes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
