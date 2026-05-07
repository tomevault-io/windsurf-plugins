---
trigger: always_on
description: This is a **Music Assistant (MA) provider plugin** for Yandex Music. It integrates with the MA server to stream music from Yandex Music and is loaded dynamically by MA at runtime.
---

# Copilot Instructions

This is a **Music Assistant (MA) provider plugin** for Yandex Music. It integrates with the MA server to stream music from Yandex Music and is loaded dynamically by MA at runtime.

## Commands

All commands use `uv run` (managed by [uv](https://docs.astral.sh/uv/)).

**Setup** (run after `git pull` too — MA models version may change):
```bash
./scripts/setup.sh
```

**Tests:**
```bash
# Unit tests only (fast, no MA server needed)
pytest tests/ -m "not integration"

# Full suite
pytest tests/

# Single test file or test
pytest tests/test_parsers.py
pytest tests/test_parsers.py::test_parse_track_with_version

# With coverage
pytest --cov=provider --cov-report=html tests/
```

**Lint / format / type-check:**
```bash
# All checks (recommended before PR)
uv run pre-commit run --all-files

# Individual tools
uv run ruff check provider/
uv run ruff format --check provider/
uv run mypy provider/
```

**Dev server** (live provider code, no Docker, port 8095):
```bash
./scripts/dev-server.sh
# UI: http://localhost:8095
```
> Do not run `pytest tests/` (full suite) while the dev server is running — port 8095 conflicts with the integration test `mass` fixture.

**Docker dev** (quick start, no local setup):
```bash
docker compose -f docker-compose.dev.yml up       # foreground
docker compose -f docker-compose.dev.yml up -d     # background
docker compose -f docker-compose.dev.yml logs -f   # follow logs
# UI: http://localhost:8095
```

## Architecture

The provider is a single Python package in `provider/` that MA loads via `manifest.json`. Source files and their responsibilities:

| File | Role |
|---|---|
| `provider.py` | `YandexMusicProvider(MusicProvider)` — main MA plugin class; implements all MA provider API methods (browse, search, library, recommendations, playback hooks) |
| `api_client.py` | `YandexMusicClient` — thin async wrapper around `yandex-music`'s `ClientAsync`; handles auth, retries, rate limiting (5 req/s), and all Yandex API calls |
| `parsers.py` | Pure functions (`parse_track`, `parse_album`, `parse_artist`, `parse_playlist`) that convert Yandex API objects into MA model objects |
| `streaming.py` | `YandexMusicStreamingManager` — resolves stream URLs, selects quality, handles AES decryption of FLAC streams |
| `auth.py` | Async functions for both login flows (`perform_qr_auth`, `perform_device_auth`) and token maintenance (`refresh_music_token`, `refresh_credentials_via_passport`, `validate_x_token`) — all delegating to the `ya-passport-auth` library |
| `constants.py` | All string constants, IDs, quality labels, locale display-name dicts, tag/category mappings |
| `__init__.py` | Provider config schema (QR auth action, token fields, quality picker, limits) and `setup()` / `get_config_entries()` |

**Data flow:** `provider.py` → `api_client.py` (fetch raw Yandex objects) → `parsers.py` (convert to MA models) → returned to MA core. Streaming is a separate path: `provider.get_stream_details()` → `streaming.get_stream_details()` → `provider.get_audio_stream()` → `streaming.get_audio_stream()`.

**Auth flow:** `__init__.py` (QR or Device Flow action clicked) → `auth.perform_qr_auth()` / `auth.perform_device_auth()` → `ya-passport-auth` library (session + polling) → returns tokens → stored in MA config. Token refresh: `provider.py` → `auth.refresh_music_token()` / `auth.refresh_credentials_via_passport()`.

**Import path in tests:** The provider is imported as `music_assistant.providers.yandex_music.*` (not via relative imports), matching how MA loads it at runtime.

## Key Conventions

### Token handling with SecretStr
- `ya-passport-auth` returns `SecretStr` (opaque wrapper); extract with `.get_secret()`
- `api_client.py` constructor takes `SecretStr`; calls `.get_secret()` only once inside `connect()`
- `provider.py` wraps config strings in `SecretStr()` before passing to `api_client` — this is a **runtime** import, not `TYPE_CHECKING`
- `api_client.py` imports `SecretStr` under `TYPE_CHECKING` (only used in type annotations; `.get_secret()` is called on the instance)
- Auth functions in `auth.py` accept `SecretStr`, return plain strings for MA config storage

### Item ID formats
- **Regular tracks:** plain `track_id` string
- **My Wave tracks:** composite `track_id@station_id` — the `@` separator (`RADIO_TRACK_ID_SEP`) encodes the rotor station for feedback; always split with `_parse_radio_item_id()` before API calls
- **Real playlists:** `owner_id:kind` (colon-separated, `PLAYLIST_ID_SPLITTER`)
- **Virtual playlists:** `my_wave` and `liked_tracks` (handled as special cases in `get_playlist` / `get_playlist_tracks`)

### Parser conventions
Parser functions in `parsers.py` follow the signature `parse_*(provider, yandex_object) -> MA model`. They receive a provider instance (for `get_item_mapping`, `instance_id`, `client.user_id`) but are otherwise pure — no async, no side effects.

### Test fixtures and snapshots
- JSON fixture files live in `tests/fixtures/{albums,artists,tracks,playlists}/` — these are real Yandex API response payloads
- Parser output is snapshot-tested via `syrupy`; snapshots live in `tests/__snapshots__/test_parsers.ambr`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trudenboy/ma-provider-yandex-music](https://github.com/trudenboy/ma-provider-yandex-music) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
