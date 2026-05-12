---
trigger: always_on
description: Dokumentace pro AI asistenty pracujici s projektem AdHUB. Pred kazdou sesi si precti tento soubor a `docs/session-directives.md`.
---

# Claude Code - AdHUB Project Guide

Dokumentace pro AI asistenty pracujici s projektem AdHUB. Pred kazdou sesi si precti tento soubor a `docs/session-directives.md`.

## Struktura projektu

```
adhub/
├── index.html              # Hlavni stranka hubu (~3200 LOC)
├── script.js               # Logika, konfigurace, preklady (~5000 LOC)
├── styles.css              # Globalni styly (~2000 LOC)
├── CLAUDE.md               # Tento soubor
├── README.md               # Dokumentace projektu
├── docs/                   # Dokumentace a vyzkum (70+ .md souboru)
│   ├── README.md           # Index dokumentace
│   ├── mcp-example.md      # MCP konfigurace
│   ├── session-directives.md # Auto-enforced pravidla pro AI sese
│   ├── research/           # Analyzy a vyzkum (31 souboru)
│   │   ├── external-services/  # Analyzy externich sluzeb (11)
│   │   └── project-research/   # Vyzkum pro projekty (20)
│   ├── artifacts/          # AI artefakty (4 + README)
│   ├── twitch-api/         # Twitch API reference (18 souboru)
│   └── kick-api/           # Kick API reference (13 souboru)
├── projects/               # 29 projektu
│   ├── youtube-downloader/ # Chrome extension, yt-dlp
│   ├── chat-panel/         # Multistream chat + WebSocket server
│   ├── cardharvest/        # Steam farming (Chrome ext + Native Host)
│   ├── paintnook/          # Digitalni malba (Canvas + TensorFlow.js)
│   ├── bg-remover/         # AI background removal (TensorFlow.js)
│   ├── pdf-editor/         # PDF editor (pdf-lib, pdf.js)
│   ├── pdf-merge/          # PDF spojovac (pdf-lib)
│   ├── pdf-search/         # PDF vyhledavani
│   ├── goalix/             # Task manager (localStorage)
│   ├── scribblix/          # Offline dokumentace (PWA)
│   ├── slidersnap/         # Before/after porovnani
│   ├── clipforge/          # Video editing (FFmpeg)
│   ├── ai-prompting/       # AI prompt engineering tools
│   ├── claude-rcs/         # Offline P2P workspace
│   ├── ip-api/             # GeoIP + IP lookup server
│   ├── ip-lookup/          # IP lookup utility
│   ├── print3d-calc/       # 3D printing calculator
│   ├── rust-calculator/    # WASM calculator
│   ├── nimt-tracker/       # Tracker app
│   ├── api-catalog/        # API aggregation reference
│   ├── adanimations/       # Animation utilities
│   ├── server-hub/         # Central hub server
│   ├── samplehub/          # Sample project templates
│   ├── betterstats/        # OBS streaming stats overlay (Twitch + Kick)
│   ├── betterytbwidget/    # YouTube Music now-playing OBS overlay
│   ├── spinning-wheel-giveaway/
│   ├── resignation-bets/
│   ├── komopizza/
│   └── video-editing-analysis/ # Research (no code)
├── server/                 # Node.js WebSocket server
└── .github/                # CI/CD workflows + analyze-projects.py
```

## Hlavni projekty

### YouTube Downloader (`projects/youtube-downloader/`)

**Typ:** Chrome Extension (Manifest V3) + Python Native Host + Node.js server

**Klicove soubory:** `plugin/manifest.json`, `plugin/content.js`, `plugin/background.js`, `plugin/popup.html`, `plugin/popup.js`, `plugin/youtube-ui.css`, `native-host/adhub_yt_host.py`

**Verze musi byt konzistentni ve vsech souborech:**
- `manifest.json`: `"version": "5.5.0"`
- `content.js`: `window.__ADHUB_YT_DL_V55__`
- `background.js`: `version: '5.5'`
- `popup.html` + `popup.js`: `v5.5`
- `native-host/adhub_yt_host.py`: `VERSION = '5.5'`

**Tok dat:** YouTube stranka → content.js (ytInitialPlayerResponse) → background.js → Zakladni rezim (chrome.downloads max 720p) NEBO Rozsireny rezim (Native Messaging → adhub_yt_host.py → yt-dlp + ffmpeg → ~/Downloads/)

### Chat Panel (`projects/chat-panel/`)

**Typ:** Web app + Node.js WebSocket server (`server/`)

### CardHarvest (`projects/cardharvest/`)

**Typ:** Chrome Extension + Native Host (Node.js) + Web UI. Farming az 32 her, 2FA, Steam CM servery. Architektura: Web UI → content.js → background.js → Native Messaging → cardharvest-host.js → steam-user → Steam CM.

### PaintNook / BG Remover (`projects/paintnook/`, `projects/bg-remover/`)

**Typ:** Canvas API + TensorFlow.js. Sdili ML modely (~212MB kazdy).

### Twitch API Documentation (`docs/twitch-api/`)

18 self-contained .md souboru. API Base URL: `https://api.twitch.tv/helix`. PubSub decommissioned 2025-04-14 — pouzivat EventSub.

| Potrebuji... | Soubor |
|---|---|
| Registrace, prvni call | `twitch-getting-started.md` |
| OAuth tokeny | `twitch-authentication.md` |
| Webhooks (HMAC) | `twitch-eventsub-webhooks.md` |
| WebSocket eventy | `twitch-eventsub-websockets.md` |
| EventSub typy | `twitch-eventsub-subscription-types.md` |
| Conduits (scaling) | `twitch-eventsub-conduits.md` |
| Chat, emotes, badges | `twitch-api-chat-whispers.md` |
| Channels, streams, raids | `twitch-api-channels-streams.md` |
| Moderace | `twitch-api-moderation.md` |
| Points, polls, predictions | `twitch-api-channel-points-polls-predictions.md` |
| Users, subscriptions | `twitch-api-users-subscriptions.md` |
| Bits, analytics | `twitch-api-bits-extensions-analytics.md` |
| Clips, videos, search | `twitch-api-clips-videos-games.md` |
| Rate limity, paginace | `twitch-api-concepts-ratelimits-pagination.md` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Deerpfy/adhub](https://github.com/Deerpfy/adhub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
