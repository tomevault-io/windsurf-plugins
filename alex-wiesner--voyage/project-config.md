---
trigger: always_on
description: Voyage is a self-hosted travel companion web application built with SvelteKit frontend and Django backend, deployed via Docker.
---

# Voyage Development Instructions

Voyage is a self-hosted travel companion web application built with SvelteKit frontend and Django backend, deployed via Docker.

**ALWAYS follow these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

## Pre-Release Policy
Voyage is **pre-release** — not yet in production use. During pre-release:
- Architecture-level changes are allowed, including replacing core libraries (e.g. LiteLLM).
- Prioritize correctness, simplicity, and maintainability over backward compatibility.
- Before launch, this policy must be revisited and tightened for production stability.

## Architecture

**Stack**: SvelteKit 2 (TypeScript) frontend · Django REST Framework (Python) backend · PostgreSQL + PostGIS database · Memcached · Docker · Bun (frontend package manager)

**Key architectural pattern — API Proxy**: The frontend never calls the Django backend directly. All API calls go to `src/routes/api/[...path]/+server.ts`, which proxies requests to the Django server (`http://server:8000`), injecting CSRF tokens and managing session cookies. This means frontend fetches use relative URLs like `/api/locations/`.

**AI Chat**: The AI travel chat assistant is embedded in Collections → Recommendations (component: `AITravelChat.svelte`). There is no standalone `/chat` route. Chat providers are loaded dynamically from `GET /api/chat/providers/` (backed by LiteLLM runtime list + custom entries like `opencode_zen`). Chat conversations stream via SSE through `/api/chat/conversations/`. Provider config lives in `backend/server/chat/llm_client.py` (`CHAT_PROVIDER_CONFIG`). Default AI provider/model saved via `UserAISettings` in DB (authoritative over browser localStorage). Chat composer supports per-provider model override via dropdown selector fed by `GET /api/chat/providers/{provider}/models/` (persisted in browser `localStorage` key `voyage_chat_model_prefs`). Collection chats inject collection UUID + multi-stop itinerary context; system prompt guides `get_trip_details`-first reasoning and confirms only before first `add_to_itinerary`. LiteLLM errors are mapped to sanitized user-safe messages via `_safe_error_payload()` (never exposes raw exception text). Invalid tool calls (missing required args) are detected and short-circuited with a user-visible error — not replayed into history. Chat agent tools (`get_trip_details`, `add_to_itinerary`) respect collection sharing — both owners and `shared_with` members can use them; `list_trips` remains owner-only. Tool outputs render as concise summaries (not raw JSON); `role=tool` messages are hidden from display and reconstructed on reload via `rebuildConversationMessages()`.

**Services** (docker-compose):
- `web` → SvelteKit frontend at `:8015`
- `server` → Django (via Gunicorn + Nginx) at `:8016`
- `db` → PostgreSQL + PostGIS at `:5432`
- `cache` → Memcached (internal)

**Authentication**: Session-based via `django-allauth`. CSRF tokens fetched from `/auth/csrf/` and passed as `X-CSRFToken` header on all mutating requests. There's also a `X-Session-Token` header path for mobile clients (handled in middleware).

## Codebase Conventions

**Backend layout**: The Django project lives in `backend/server/`. Apps are `adventures` (core: locations, collections, itineraries, notes, transportation), `users`, `worldtravel` (countries/regions), `integrations`, `achievements`, and `chat` (LLM chat agent with dynamic provider catalog). Views inside `adventures` are split into per-domain files under `adventures/views/` (e.g. `location_view.py`, `collection_view.py`).

**Backend patterns**:
- DRF `ModelViewSet` subclasses for all CRUD resources; custom actions with `@action`
- `get_queryset()` always filters by `user=self.request.user` to enforce ownership
- `GenericForeignKey` (`content_type` + `object_id`) used in `CollectionItem` to allow collections to contain heterogeneous items
- Background geocoding runs in a daemon thread via `threading.Thread`
- Money fields use `djmoney` (`MoneyField`); geospatial queries use PostGIS via `django-geojson`

**Frontend layout**: Source lives in `frontend/src/`. Pages are in `src/routes/` (file-based SvelteKit routing). Shared code lives in `src/lib/`:
- `src/lib/types.ts` — all TypeScript interfaces (`Location`, `Collection`, `User`, `Visit`, etc.)
- `src/lib/index.ts` — general utility functions
- `src/lib/index.server.ts` — server-only utilities (used in `+page.server.ts` and `+server.ts` files)
- `src/lib/components/` — Svelte components organized by domain (`locations/`, `collections/`, `map/`, `cards/`, `shared/`); includes `AITravelChat.svelte` for Collections chat
- `src/locales/` — i18n JSON files (uses `svelte-i18n`); wrap all user-visible strings in `$t('key')`

**Frontend patterns**:
- Use `$t('key')` from `svelte-i18n` for all user-facing strings; add new keys to locale files
- API calls always use `credentials: 'include'` and the `X-CSRFToken` header (get token from the `csrfToken` store/prop)
- Reactive state updates require reassignment to trigger Svelte reactivity: `items[i] = updated; items = items`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Alex-Wiesner) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
