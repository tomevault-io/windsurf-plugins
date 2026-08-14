---
trigger: always_on
description: **Project**: AstrBot plugin integrating Suwayomi-Server for manga search, reading, chapter packaging/download, and subscription updates.
---

# AGENTS.md

## Quick Reference

**Project**: AstrBot plugin integrating Suwayomi-Server for manga search, reading, chapter packaging/download, and subscription updates.
**Language**: Python 3.12+ | **Package manager**: uv | **Framework**: AstrBot plugin system

## Documentation

- [贡献指南](CONTRIBUTING.md) — 开发环境搭建、开发流程、提交规范、添加新命令
- [开发指南](docs/dev/development.md) — 架构详解、设计决策、数据流
- [Suwayomi API 参考](docs/dev/suwayomi-api.md) — GraphQL API 文档
- [配置教程](docs/setup.md) — Suwayomi-Server 部署和插件配置
- [变更日志](CHANGELOG.md) — 版本更新记录
- [文档更新清单](docs/dev/doc-update-checklist.md) — 各类变更需同步更新的文件列表

## Commands

```bash
# Unit tests (no network needed)
uv run pytest tests/test_pack.py tests/test_models.py tests/test_client.py tests/test_subscription.py tests/test_web_api.py tests/test_batch_subscribe.py tests/test_push.py tests/test_service.py tests/test_ai_service.py tests/test_ai_tools.py -v

# Integration tests (requires live Suwayomi-Server)
uv run pytest tests/test_live_api.py tests/test_live_web_api.py -v -s
# Custom server: SUWAYOMI_URL=http://host:4567 uv run pytest tests/test_live_api.py tests/test_live_web_api.py -v -s
# Note: live tests auto-skip (3s probe) when the server is unreachable, so plain `uv run pytest` is always green without a server.

# All tests
uv run pytest -v

# Syntax check
python -c "import ast; ast.parse(open('main.py', encoding='utf-8').read()); print('OK')"
```

## Architecture

```
main.py (SuwayomiPlugin — thin dispatch layer)
  ├── suwayomi/client.py (SuwayomiClient - async GraphQL HTTP)
  ├── suwayomi/models.py (Source, Manga, Chapter, SearchResult dataclasses)
  ├── suwayomi/service.py (resolve_manga, resolve_chapter, get_or_fetch_chapters, fmt helpers)
  ├── suwayomi/ai_service.py (structured, side-effect-free Agent search/chapter/subscription service)
  ├── suwayomi/ai_tools.py (FunctionTool schemas and registration factory)
  ├── suwayomi/updater.py (check_updates, run_update_loop)
  ├── utils/downloader.py (download_one, download_images, fetch_pages_local)
  ├── utils/pack.py (pack_zip, pack_cbz, pack_pdf — image packaging)
  ├── utils/pusher.py (push_chapter_images, push_chapter_file, schedule_cleanup)
  ├── utils/subscription.py (SubscriptionManager - AstrBot KV storage)
  ├── web/api.py (WebUI API handlers — standalone functions, dependency-injected)
  └── pages/dashboard/ (WebUI: 仪表盘 + 订阅管理 + 配置)
```

- `main.py`: Plugin entry, all commands under `@filter.command_group("漫画")`, six AstrBot Agent tools, background update loop, WebUI API registration. Thin dispatch layer — all business logic delegated to service/updater/downloader/pusher modules.
  - `suwayomi/client.py`: All Suwayomi interaction via `POST /api/graphql`; supports none/basic/jwt auth. Exposes `auth_headers` property for image download auth.
- `suwayomi/models.py`: Pure dataclasses with `from_dict()` factory methods
- `suwayomi/service.py`: Business logic — manga/chapter resolution, chapter fetching/caching, text normalization, status emoji mapping. All functions are standalone with dependency-injected parameters (client, sub_mgr, get_kv_data, etc.)
- `suwayomi/ai_service.py`: Structured AI-facing search, chapter lookup, subscribe/unsubscribe, and subscription listing. Returns stable manga/chapter IDs and never sends messages.
- `suwayomi/ai_tools.py`: Explicit JSON Schemas for six tools — `suwayomi_search_manga`, `suwayomi_get_chapters`, `suwayomi_send_chapter`, `suwayomi_subscribe_manga`, `suwayomi_get_subscriptions`, `suwayomi_unsubscribe_manga` — registered through `context.add_llm_tools()`; custom `call()` dispatch keeps event binding stable during initial load and config-driven re-registration.
- `suwayomi/updater.py`: Update engine — `check_updates()` scans all subscriptions for new chapters (parallel, `_UPDATE_CONCURRENCY=5` Semaphore), pushes notifications, triggers auto-push, records `suwayomi_last_update_check` timestamp. `run_update_loop()` is the background task wrapper. Imported by `main.py` with pre-bound push callbacks.
- `utils/downloader.py`: Image download pipeline — `download_one()` with exponential backoff, `download_images()` parallel batch download (accepts `headers` for auth), `fetch_pages_local()` downloads chapter pages to temp dir (passes `client.auth_headers`).
- `utils/pack.py`: Pack images into ZIP, CBZ, or PDF files; `parse_download_args()` for command arg parsing; shared helpers `sanitize_filename()`, `normalize_pack_format()`, `build_chapter_output_path()`, `pack_images()` used by download/push/AI-send.
- `utils/pusher.py`: Push delivery — `push_chapter_images()` sends images inline or via forward, `push_chapter_file()` sends packaged file. Also exports `schedule_cleanup()` for delayed temp dir cleanup (tracked in `_cleanup_tasks`, cancelled via `cancel_pending_cleanups()` on terminate), `is_aiocqhttp_target()` for platform detection, and `build_image_chain()` — the single shared builder for image/forward message chains (read, auto-push, AI send).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FFFold/astrbot_plugin_suwayomi_server](https://github.com/FFFold/astrbot_plugin_suwayomi_server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
