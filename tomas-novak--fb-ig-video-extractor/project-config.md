---
trigger: always_on
description: A Telegram bot that takes a Facebook Reels or Instagram Reels URL, automatically
---

# CLAUDE.md — FB/IG Video Extractor

## What this project does

A Telegram bot that takes a Facebook Reels or Instagram Reels URL, automatically
pulls the video content, transcribes the audio, extracts the place and saves the
metadata to Google Sheets.

Live demo of the map (static sample data, no bot required):
https://tomas-novak.github.io/fb-ig-video-extractor/ (source: `docs/index.html`).

## Architecture

```
User (phone)
  → sends a URL to the Telegram bot
      → Python API (VPS)
          → yt-dlp downloads the VIDEO (not audio – FB offers no audio-only stream to datacenters)
          → Gemini Flash analyzes the video: audio transcript + on-screen text + caption (one call)
          → Google Sheets API saves a row
          → Telegram Bot API replies to the user
```

Note: we send Gemini the whole video (not extracted audio). The reason: on a
datacenter IP Facebook does not offer a separate audio stream, only video. On top
of that Gemini reads the text shown in the video, which makes the place
determination more accurate.

## Stack

- **Runtime**: Python 3.10+ (verified on 3.10.12 on the production VPS; no construct in the code requires 3.11)
- **Web framework**: FastAPI + uvicorn
- **Telegram**: python-telegram-bot or direct Bot API calls
- **Video download**: yt-dlp (format `hd/sd/best`)
- **AI (video analysis)**: Google Gemini 2.5 Flash (multimodal video – audio + image)
- **Database**: Google Sheets (google-auth + gspread)
- **Hosting**: own VPS (Ubuntu 22.04, systemd + Caddy)

## Key files

```
FB_IG_video_extractor/
├── CLAUDE.md          # this file
├── README.md          # user documentation (English, default)
├── README.cs.md       # user documentation (Czech)
├── main.py            # FastAPI app + Telegram webhook handler
├── extractor.py       # yt-dlp download logic
├── analyzer.py        # Gemini API calls (transcription + analysis)
├── i18n.py            # bot language (BOT_LANGUAGE) + categories (CATEGORIES) + texts
├── sheets.py          # Google Sheets writing
├── models.py          # data models (VideoMetadata)
├── requirements.txt
├── deploy/            # systemd unit, Caddyfile, update and DuckDNS scripts
├── docs/deploy-vps.md # general guide for deploying to your own VPS (English)
├── docs/index.html    # static public demo of the map (GitHub Pages), sample data only
├── railway.toml       # Railway build/deploy config
├── nixpacks.toml      # Railway: adds ffmpeg to the auto-detected Python
├── gen_railway_env.py # helper script: .env -> railway-variables.local.txt
└── .env.example       # environment variables template
```

## Environment variables

Full reference with descriptions: `.env.example` (kept as the single source
of truth rather than duplicated here).

The three required for any deployment: `TELEGRAM_BOT_TOKEN`, `GEMINI_API_KEY`,
`GOOGLE_SHEETS_ID` (+ `GOOGLE_SERVICE_ACCOUNT_FILE` for local dev or
`GOOGLE_SERVICE_ACCOUNT_JSON` for servers — see `sheets.py`). `PUBLIC_URL`/
`HOST` matter specifically for deployment - see below.

## Google Sheets structure (Sheet1)

| A: Date | B: URL | C: Author | D: Title | E: Place | F: Lat | G: Lng | H: Category | I: Tags | J: Summary | K: Transcript | L: Source | M: group_id |

**M: group_id** — places with the same group_id are shown on the map as a single
pin with multiple videos. Merging is proposed by Claude Haiku (the dedup command
in Telegram → Merge/Keep buttons). See `dedup.py`. Merging deletes nothing and is
reversible (clear the group_id in the sheet).

## Supported URL formats

- `https://www.facebook.com/reel/ID`
- `https://www.instagram.com/reel/ID`
- `https://www.instagram.com/p/ID`
- `https://www.tiktok.com/@user/video/ID`
- `https://www.youtube.com/shorts/ID`
- Share links (shortened: `fb.watch`, `vm.tiktok.com`, `youtu.be`) — yt-dlp expands them automatically

## Categories (for filtering on the map)

Default (en): `swimming` · `hiking` · `food` · `culture` · `nature` · `sport` · `fun` · `hotel` · `other`

Default (cs): `koupání` · `turistika` · `jídlo` · `kultura` · `příroda` · `sport` · `zábava` · `hotel` · `jiné`

Categories are configurable through the `CATEGORIES` env variable (see `i18n.py`);
the last one in the list is the fallback. The language of the bot's replies, the
map and Gemini summaries is driven by `BOT_LANGUAGE` (`en` default / `cs`); the
audio transcript stays in the language of the video.

## Development

```bash
python -m venv venv
venv\Scripts\activate       # Windows
pip install -r requirements.txt
cp .env.example .env        # fill in the values
uvicorn main:app --reload --port 8000
```

## Language policy

The repository is public and English is its working language.

- Write commit messages, branch names and PR text in **English**.
- Code comments, docstrings and development documentation (this file) are in **English**.
- Czech stays only where it is product data, not prose:
  - `README.cs.md` (Czech user documentation)
  - the `cs` message catalog, command names and categories in `i18n.py`
  - the `cs` Gemini prompt templates in `analyzer.py`
  - the `T_cs` map UI dictionary in `map_page.py`
  - test fixtures and assertions, since `tests/conftest.py` pins `BOT_LANGUAGE=cs`
    to exercise the Czech path

## Deployment


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomas-novak/fb-ig-video-extractor](https://github.com/tomas-novak/fb-ig-video-extractor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
