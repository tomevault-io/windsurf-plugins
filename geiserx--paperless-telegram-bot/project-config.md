---
trigger: always_on
description: Full-featured Telegram bot for managing Paperless-NGX documents entirely through chat -- upload files, search the archive with full-text search, manage metadata (tags, correspondents, document types), review inbox, and download documents.
---

# CLAUDE.md — paperless-telegram-bot

## Overview
Full-featured Telegram bot for managing Paperless-NGX documents entirely through chat -- upload files, search the archive with full-text search, manage metadata (tags, correspondents, document types), review inbox, and download documents.

## Tech Stack
- Python 3.11+
- python-telegram-bot (Telegram Bot API)
- httpx (async HTTP client for Paperless-NGX API)
- FastAPI + uvicorn (health check endpoint)
- Docker (non-root container)
- ruff (linter + formatter)
- pytest + pytest-asyncio + respx (testing)
- pre-commit hooks
- PyPI package (`paperless-telegram-bot`)
- GitHub Actions CI

## Development
```bash
pip install -r requirements.txt
python -m src.paperless_bot      # Run the bot

# Lint and format
ruff check src/
ruff format src/

# Tests
python3 -m pytest tests/ -v

# Pre-commit
pre-commit run --all-files
```

## Architecture
- `src/paperless_bot/bot/handlers.py` — Telegram bot logic, command handlers, callback routing, inbox review flow
- `src/paperless_bot/api/client.py` — Paperless-NGX API client, caching, inbox tag auto-detection
- `src/paperless_bot/bot/keyboards.py` — Inline keyboard builders (metadata, tag selection, inbox with review buttons)
- `src/paperless_bot/config.py` — Environment variables and configuration
- `src/paperless_bot/__main__.py` — Entry point, health server, CLI
- `tests/` — pytest test suite (`test_client.py` and `test_config.py` are maintained; `test_handlers.py` is stale)
- `docs/ROADMAP.md` — Planned features and future improvements
- `Dockerfile` — Non-root Docker container
- `pyproject.toml` — Package metadata and build config
- `.env.example` — Required environment variables template

## Git Workflow
- Create feature branches and submit pull requests
- Do NOT commit directly to main branch
- Descriptive branch names (e.g., `feat/batch-upload`, `fix/search-pagination`)
- Squash merge PRs to keep main history clean

## Deployment

### Versioning Rules
- **NEVER** reuse or force-retag an existing version. Tags only go forward.
- **Patch** (`v0.6.0` -> `v0.6.1`): Bug fixes, small tweaks
- **Minor** (`v0.6.x` -> `v0.7.0`): New features, significant behavior changes
- **Major** (`v0.x.y` -> `v1.0.0`): Breaking changes
- Check the latest tag before tagging: `git describe --tags --abbrev=0`
- Update version in both `pyproject.toml` and `src/paperless_bot/__init__.py`
- Also update the image tag in `gitea/geiserback/paperless-telegram-bot/docker-compose.yml` to match

### Release Steps
1. Create feature branch, push, open PR, squash-merge to `main`
2. Create a GitHub release via `gh release create vX.Y.Z --target main` (GHA triggers on release tags)
3. **Wait for GHA `docker-publish.yml` to complete** -- verify the run succeeds before proceeding
4. Update `gitea/geiserback/paperless-telegram-bot/docker-compose.yml` with the new tag, commit and push to Gitea
5. Redeploy via Portainer API on **geiserback** (stack ID `80`, endpoint `2`)
6. Verify with `docker ps --filter name=paperless_telegram_bot` and check logs for `Bot commands registered`

## Environment Variables

### Required

| Variable | Description |
|----------|-------------|
| `TELEGRAM_BOT_TOKEN` | Telegram Bot API token |
| `PAPERLESS_URL` | Internal Paperless-NGX URL (e.g., `http://192.168.10.110:8000`) |
| `PAPERLESS_TOKEN` | Paperless-NGX API token |

### Optional

| Variable | Default | Description |
|----------|---------|-------------|
| `TELEGRAM_ALLOWED_USERS` | *(empty = open)* | Comma-separated Telegram user IDs allowed to use the bot |
| `PAPERLESS_PUBLIC_URL` | `PAPERLESS_URL` | User-facing URL for clickable document links (e.g., Tailscale hostname) |
| `MAX_SEARCH_RESULTS` | `10` | Number of results per page in search/recent/inbox |
| `REMOVE_INBOX_ON_DONE` | `true` | Remove inbox tag when user clicks "Done" in metadata flow. Set `false` to disable |
| `INBOX_TAG` | *(auto-detect)* | Explicit inbox tag name override. If unset, auto-detects the tag with `is_inbox_tag=true` from Paperless API |
| `LOG_LEVEL` | `INFO` | Logging level (DEBUG, INFO, WARNING, ERROR) |
| `HEALTH_PORT` | `8080` | Port for the `/health` HTTP endpoint |

## Key Architecture Decisions

### Bot Commands
Registered via `set_my_commands` in `post_init` callback: `/search`, `/recent`, `/inbox`, `/stats`, `/help`.

### Callback Data Encoding
Telegram's `callback_data` has a **64-byte limit**. All callback prefixes are kept short:

| Prefix | Purpose |
|--------|---------|
| `meta:tags:`, `meta:corr:`, `meta:dtype:`, `meta:done:` | Metadata menu actions |
| `tag:x:`, `tag:o:` | Tag toggle (checked/unchecked) |
| `tagp:`, `tagok:` | Tag pagination, confirm |
| `newtag:`, `newcorr:`, `newdtype:` | Create new metadata item |
| `ccr:` | Cancel create |
| `corr:`, `corrp:` | Correspondent select, paginate |
| `dtype:`, `dtypep:` | Document type select, paginate |
| `dl:` | Download document |
| `sp:` | Search pagination |
| `rev:` | Mark as reviewed (remove inbox tag) |

Search queries are stored in a per-chat dict (`search_queries`) since they'd exceed the 64-byte limit.

### Inbox Workflow

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GeiserX/paperless-telegram-bot](https://github.com/GeiserX/paperless-telegram-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
