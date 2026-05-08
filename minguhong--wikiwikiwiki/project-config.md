---
trigger: always_on
description: This file is the single execution contract for LLMs/agents working on WikiWikiWiki.
---

# AGENTS.md

This file is the single execution contract for LLMs/agents working on WikiWikiWiki.

## 0) Product Summary

WikiWikiWiki is a file-based wiki for personal or small-team use. It uses no database and stores primary content in plain text files (`content/*.txt`). The runtime entry point is a single `index.php` request flow. Reading can be anonymous, while write/admin actions are controlled by authentication and role checks.

## 1) Working Principles

Use these principles for every decision:

- Prefer simpler behavior over feature breadth.
- Keep operations obvious (install, backup, restore, update).
- Keep patterns consistent across files and flows.
- Optimize for recovery and clarity, not complexity.
- Ship small, reversible changes.

If a change needs long explanation, reduce scope.

## 2) Runtime Overview

Boot sequence:

1. `index.php`
2. `wikiwikiwiki/src/bootstrap.php`
3. `wikiwikiwiki/src/load.php`
4. `wikiwikiwiki/src/router.php`

Core constants (from `bootstrap.php`):

- `WIKI_ROUTE_PATH = 'wiki'`
- `PAGE_LIST_LIMIT = 20`
- `ALL_PAGES_LIMIT = 100`
- `FEED_LIMIT = 10`
- `CACHE_FILE_MAX_BYTES = 5 * 1024 * 1024` (5 MB)
- `PHP_REQUIRED_VERSION = '8.2.0'`

Theme name validation:

- Regex: `^[A-Za-z0-9_-]{1,64}$`

### 2.1 Runtime Config Keys (`config/config.php`)

Use this as the canonical key set for settings-related work:

| Key | Default/Fallback Behavior | Notes |
| --- | --- | --- |
| `WIKI_TITLE` | missing key falls back to `'WikiWikiWiki'` | Wiki name in header and metadata |
| `WIKI_DESCRIPTION` | missing key falls back to `'A wiki made with WikiWikiWiki'` | Used in meta/RSS description |
| `BASE_URL` | `''` -> auto-detect from request | Must be `http(s)` URL when set |
| `THEME` | `''` (no theme) | Must match available theme directory |
| `LANGUAGE` | missing key falls back to resolved i18n language from available catalogs (commonly `ko`/`en`); when no catalogs exist, falls back to `en` | Must exist in language catalog when configured |
| `TIMEZONE` | missing key falls back to `''` (server timezone); invalid value falls back to `UTC` | Must be valid PHP timezone when set |
| `HOME_PAGE` | missing key falls back to `'Welcome'` | Must pass title sanitization and filename limit |
| `EDIT_PERMISSION` | `'private'` | Allowed values: `'private'`, `'public'`, `'fully_public'` |

## 3) Repository Map (What Matters at Runtime)

- Entry: `index.php`
- Engine code: `wikiwikiwiki/src/`
- Engine templates: `wikiwikiwiki/templates/`
- Engine assets: `wikiwikiwiki/assets/`
- Engine i18n: `wikiwikiwiki/i18n/`
- Config: `config/config.php`
- Theme overrides: `themes/<theme>/{templates,assets,i18n}/`
- Page source of truth: `content/*.txt`
- History snapshots: `history/*.txt`
- User data: `users/*.json`
- Auth state files: `users/.rate_limits.json`, `users/.remember_tokens.json`
- Derived indexes/cache: `cache/*.json`

## 4) Non-Negotiable Invariants

### 4.1 Storage and locking

- Required writable dirs: `content`, `history`, `users`, `cache`.
- Global lock file path: `/.lock` (`wiki_lock_path()`).
- State-changing file operations must run inside `wiki_with_lock(...)`.
- `wiki_with_lock(...)` is reentrant in-process: nested calls execute callbacks under the outer lock without acquiring another lock; nested `shared` mode does not downgrade/upgrade lock state.
- File writes must be atomic via `file_put_atomic(...)`.
- If storage health fails at boot, app must fail fast with HTTP 503 (`fail_fast_on_storage_unavailable()`).

### 4.2 Auth and permissions

- Read is allowed to anonymous users.
- `EDIT_PERMISSION === 'private'` and `EDIT_PERMISSION === 'public'`: create/edit/delete require login.
- `EDIT_PERMISSION === 'fully_public'`: create/edit/delete are allowed without login.
- Admin-only operations must use `require_admin()`.
- History restore (`action=restore` on `/wiki/{title}/history/{timestamp}`) requires `require_admin()`.
- `delete_history=1` on page delete is honored only for admins; non-admin requests must ignore the flag and keep history files.
- Browser form POST actions must pass CSRF (`validate_post_request()`).
- `EDIT_PERMISSION === 'public'` or `EDIT_PERMISSION === 'fully_public'` enables registration (`/register`) and enables POST honeypot checks.
- `EDIT_PERMISSION === 'private'` disables registration and skips honeypot checks.
- Anonymous edits are allowed only by `EDIT_PERMISSION === 'fully_public'`; history author for anonymous edits remains empty and UI renders `history.unknown_editor`.
- Authenticated sessions must bind `username`, `role`, and `user_fingerprint`; legacy sessions without `user_fingerprint` must not be upgraded in place.
- Optional remember-login is browser-specific and tied to the authenticated user fingerprint; logout revokes the current remember token, while password change, self-delete, and admin-side user delete must revoke all remember tokens for that user before success.
- Do not store raw client IP in history filenames.
- Install-first rule is mandatory:
  - If `user_count() === 0`, redirect every path except `/install` to `/install`.

### 4.3 Data model

- `content/*.txt` is canonical data.
- `cache/*.json` is disposable derived state and must be rebuildable.
- Corrupt/stale cache must degrade to rebuild, not fatal.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [minguhong/WikiWikiWiki](https://github.com/minguhong/WikiWikiWiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
