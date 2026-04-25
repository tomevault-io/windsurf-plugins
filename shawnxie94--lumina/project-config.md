---
trigger: always_on
description: **Generated:** 2026-03-25 20:11 Asia/Shanghai
---

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-03-25 20:11 Asia/Shanghai
**Commit:** 9d04825
**Branch:** main

## OVERVIEW
Lumina is a content workspace with a Next.js 14 frontend (pages router), FastAPI backend, and WXT extension for web capture, AI reading, RSS subscription, comments, and content operations workflows.

## STRUCTURE
```
./
├── backend/              # FastAPI app, models, worker, migrations
├── frontend/             # Next.js pages router app (web UI + API routes)
├── extension/            # WXT browser extension
├── docs/                 # Product docs and screenshots
├── scripts/              # Repo-level scripts (docker healthcheck, etc.)
└── data/                 # SQLite database + media volume
```

## WHERE TO LOOK
| Task | Location | Notes |
|------|----------|-------|
| Backend app entry | `backend/main.py` `backend/app/main.py` | `main.py` exports app; app factory in `backend/app/main.py` |
| Backend API routers | `backend/app/api/routers/` | Routers split by domain, URLs unchanged |
| Backend router wiring | `backend/app/api/router_registry.py` | Routers are mounted under `/backend/api/*` only |
| Backend dependencies/middleware | `backend/app/core/dependencies.py` `backend/app/core/http.py` | Shared auth/internal/cors/request-id logic |
| Backend runtime settings | `backend/app/core/settings.py` | Central env loading + startup fail-fast validation |
| Backend settings reference | `backend/docs/runtime-settings.md` | Runtime env defaults and validation constraints |
| Backend domain services | `backend/app/domain/` | Query/command/AI/task split |
| Backend language heuristic | `backend/ai_client.py` | `is_english_content` uses cleaned-text ratio + Han-char guard |
| Backend recommendation refresh API | `backend/app/api/routers/settings_router.py` | Admin batch endpoint for recommendation embedding refresh |
| Backend embedding batch logic | `backend/app/domain/article_embedding_service.py` | Rebuild queue with model/hash skip logic |
| Backend RSS generation | `backend/app/domain/article_rss_service.py` | Public RSS feed assembly + cache key strategy |
| Backend backup import/export | `backend/app/api/routers/backup_router.py` `backend/app/domain/backup_service.py` | JSON backup stream export + incremental restore |
| Backend tag APIs | `backend/app/api/routers/tag_router.py` `backend/app/domain/article_tag_service.py` | Shared tag listing, regenerate, and orphan cleanup logic |
| Backend DB migrations | `backend/alembic/` `backend/scripts/migrate_db.py` | Alembic-based schema/index upgrade path |
| Backend unit tests | `backend/tests/unit/` | Pytest-based unit tests for core/domain/utils modules |
| Route contract baseline | `backend/scripts/route_contract_baseline.json` | API signature regression baseline for modular routers |
| Response contract baseline | `backend/scripts/response_contract_baseline.json` | Key API response shape regression baseline |
| DB models + init | `backend/models.py` | Models + DB setup + defaults |
| AI worker loop | `backend/worker.py` | Background task processor |
| Frontend home page | `frontend/pages/index.tsx` | Hero + latest-article cards |
| Frontend list page | `frontend/pages/list.tsx` | Filters, batch ops, pagination |
| Frontend detail page | `frontend/pages/article/[id].tsx` | AI panels, polling, TOC, notes, highlight annotations, tag edits |
| Frontend admin settings | `frontend/pages/admin.tsx` | Model/prompt/admin config UI |
| Frontend login/setup | `frontend/pages/login.tsx` | Admin setup + login gate |
| Frontend shared app header | `frontend/components/AppHeader.tsx` | Theme/language switch, notification center, RSS entry |
| Frontend comment auth API | `frontend/pages/api/auth/[...nextauth].ts` | OAuth provider config from backend settings |
| Frontend comment proxy API | `frontend/pages/api/comments/[articleId].ts` | Reads posts comments via Next API route |
| Frontend API client | `frontend/lib/api.ts` | Axios base + typed exports |
| Frontend i18n dictionary | `frontend/lib/i18n.ts` | zh-CN/en string map used across pages |
| Frontend notification store | `frontend/lib/notifications.ts` | Local persisted task/API/system notifications |
| Frontend markdown rendering | `frontend/lib/safeHtml.ts` | Unified pipeline with GFM + math + sanitize + media embeds |
| Extension API client | `extension/utils/api.ts` | Fetch wrapper + auth headers |
| Extension popup UI | `extension/entrypoints/popup/main.js` | Main capture flow |
| Extension extraction | `extension/entrypoints/content.ts` `extension/utils/siteAdapters.ts` | Content parsing/adapters + formula-preserving fallback for Readability |
| Extension shared helpers | `extension/utils/` | History/error/i18n/markdown helper modules |
| Product docs | `docs/trd/trd-optimized.md` `docs/trd/trd-minimal.md` | TRD baselines |

## CODE MAP
| Symbol | Type | Location | Role |
|--------|------|----------|------|
| HomePage | Function | `frontend/pages/index.tsx` | Landing page + latest content feed |
| Home | Function | `frontend/pages/list.tsx` | Article list page controller |
| AppHeader | Function | `frontend/components/AppHeader.tsx` | Global header with notifications, RSS, theme, and language controls |
| PopupController | Class | `extension/entrypoints/popup/main.js` | Extension popup UI logic |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shawnxie94/lumina](https://github.com/shawnxie94/lumina) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
