---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

A self-hosted Telegram bot that downloads media via `yt-dlp` and uploads it back to the user. Pure utility — no LLM calls. Built on **aiogram 3.x** (async). Targets homelab/Docker deployment.

## Run / build / test

```bash
# Docker (primary path: bot + local Telegram Bot API server, 2GB uploads)
docker compose up -d --build       # build + start both services
docker compose logs -f bot         # follow bot logs
docker compose up -d bot           # restart bot ONLY after .env change (no rebuild)
docker compose up -d --build bot   # rebuild bot after CODE change (see gotcha below)
docker compose down                # stop

# Bare Python (standard API, 50MB limit, supports Firefox cookies)
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt
python main.py

# Tests (pytest + pytest-asyncio)
pip install -r requirements-dev.txt && pytest
# …or without a venv:
uv run --with pytest --with pytest-asyncio --with aiogram --with structlog \
       --with python-dotenv --with aiohttp pytest

# Syntax check
python -m py_compile main.py src/**/*.py
```

Tests live in `tests/` and cover the pure-logic units (config parsing, sanitization, URL extraction, platform detection, yt-dlp command building, caption builder, allowlist middleware, queue, ffmpeg thumbnail resizing). They use **no network or Telegram**: `get_info`/downloads are mocked, and `tests/conftest.py` patches out the `yt-dlp --version` probe and sets a dummy `BOT_TOKEN`. `pytest.ini` sets `asyncio_mode = auto`, so `async def test_*` functions run directly. The ffmpeg-dependent thumbnail tests self-skip when ffmpeg is absent. After non-trivial changes, run pytest; for end-to-end behaviour also exercise the bot in Telegram or via `docker compose exec -T bot python -c "..."`.

## Architecture

Request flow: Telegram → `dp` (aiogram Dispatcher) → `auth_middleware` → handler → queue → `yt-dlp` → uploader → cleanup.

- `main.py` — entry point. Builds `Bot`/`Dispatcher`, wires graceful shutdown, starts polling. Chooses local-API-server session vs. standard API based on `API_SERVER_URL`.
- `src/bot/router.py` — `create_router()` builds the `Dispatcher`, registers the **auth middleware** (`outer_middleware` on `dp.message` and `dp.callback_query`), all command/text handlers, and the `q:`-prefixed callback for the inline quality picker.
- `src/bot/quality.py` — inline quality-picker choices (`QUALITY_CHOICES`) and `quality_params()` mapping each to `(MediaFormat, max_height)`. `/formats` renders these as buttons; `BotHandlers.on_quality_choice` resolves the tapped token (`stash_url`/`_pending`) and calls `enqueue_download`.
- `src/bot/handlers.py` — `BotHandlers`: URL extraction, queueing, the background download→upload→cleanup task, status-message editing. `handle_message()` (the catch-all `@dp.message()` handler in `router.py`) processes URLs regardless of chat type, but the "send me a URL" nudge for URL-less messages only fires in private chats or when the bot is directly addressed in a group (`_is_addressed()`: a reply to the bot's own message, or an `@mention` — bot username is fetched once via `get_me()` and cached in `_bot_username`) — otherwise ordinary group chatter is ignored silently.
- `src/commands/handlers.py` — `CommandHandlers`: `/start /help /audio /video /cancel /status /formats`.
- `src/config/settings.py` — `Settings` dataclass loaded once from env (`get_settings()` singleton). All config flows through here.
- `src/utils/logger.py` — `StructuredLogger`: stdout always; when `LOG_FILE` is set, also a `RotatingFileHandler` (10MB × 10) for a durable download record. Handler setup is **idempotent** — modules call `get_logger()` at import time (stdout only) before `main()` re-runs `setup_logging()` with the file path, so handlers are added if-missing rather than all-or-nothing. In Docker, `LOG_FILE=/data/tg-media-bot.log` on the `bot-logs` volume persists across rebuilds.
- `src/downloaders/ytdlp.py` — `YtDlpDownloader`: subprocess wrapper around `yt-dlp`. Platform detection, format selection. `AUDIO_ONLY_PLATFORMS` (e.g. soundcloud) are forced to audio via `_effective_format()` regardless of the user's preference. Audio downloads embed cover art + tags (`--embed-thumbnail/--embed-metadata`) and write `--write-info-json`; `_find_thumbnail()` / `_read_info_json()` surface cover art, title, artist, and duration on `DownloadResult`. The output template caps the title at `_TITLE_MAX_BYTES` (`%(title).150B`) so long titles can't hit `[Errno 36] File name too long`. Video downloads add `--recode-video mp4` + `-movflags +faststart` so direct-file URLs (e.g. `.webm`) come out as Telegram-streamable MP4. **Geo-block fallback:** if a download fails and the stderr matches `_GEO_MARKERS` (country/region licensing block) *and* `PROXY_URL` is set, it retries **once** through `--proxy` (intended for CIS-licensed media via a regional exit). The proxy is *only* used on this fallback, never on the first attempt.
- `src/types/download.py` — shared domain types used across modules: `DownloadTask`, `DownloadStatus` enum (queued→downloading→processing→uploading→completed/failed/cancelled), `MediaFormat` enum (`video`/`audio`/`auto`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antlis/tg-media-bot](https://github.com/antlis/tg-media-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
