---
trigger: always_on
description: Frontend (React 19 + Vite)  ←→  FastAPI /api/v1/  ←→  SQLite (WAL)
---

# AGENTS.md

## Architecture at a Glance

```
Frontend (React 19 + Vite)  ←→  FastAPI /api/v1/  ←→  SQLite (WAL)
                                      ↓
                              Background Worker
                                      ↓
                    ┌─────── 6-Stage Monitoring Pipeline ───────┐
                    │ 1. Context Build (crawl + 3-role AI debate)│
                    │ 2. Signal Collect (SEO/GEO/Community/SERP) │
                    │ 3. Signal Normalize                        │
                    │ 4. Domain Review (4 AI analysts)           │
                    │ 5. Strategy Synthesis                      │
                    │ 6. Persist & Publish                       │
                    └────────────────────────────────────────────┘
```

## Phase 1 Repositioning (2026-04-28)

The public site (`aidcmo.com`) has been repositioned from **"B2B email leads + SEO/GEO services"** to **"open-source AI growth tools, audits delivered by humans"**. Branch: `feat/repositioning-phase-1` (commit `17d9208`).

### What changed

- **Public routes**: `/services` (skeleton) + `/hosted` (waitlist) are new. The old `/b2b-leads`, `/sample-data`, `/data-policy`, `/seo-geo` routes are **deleted from the SPA** and 301-redirected server-side (locale-preserving, GET + HEAD).
- **`PublicServicePage` kinds reduced from 6 → 2**: only `open-source` and `contact` remain. Type union narrowed in `frontend/src/pages/PublicServicePage.tsx`.
- **i18n**: `service.b2b.*`, `service.sample.*`, `service.policy.*`, `service.seoGeo.*` namespaces deleted across 5 locales. New: `landing.builtInOpen.*`, `landing.hosted.*`, `service.audit.*`. `landing.hero*` / `landing.meta*` rewritten. ja/ko/es rely on auto EN-fallback (no new keys added there).
- **Path helpers** in `frontend/src/content/marketing.ts`: `getB2BLeadsPath` / `getSampleDataPath` / `getDataPolicyPath` / `getSeoGeoPath` are now `@deprecated` wrappers that return `/services` or `/`. Slated for removal once any remaining callers in non-marketing surfaces are gone (grep `getB2B|getSampleData|getDataPolicy|getSeoGeo`).
- **`PublicSiteHeader` primary CTA**: target switched from `/sample-data` → `/services`.
- **Server-side SEO** (`web/app.py`): `_HOME_STATIC_SITE_COPY_BY_LOCALE`, `_SERVICE_PAGE_METADATA_BY_PATH`, `_build_home_json_ld`, and home-route metadata literals all rewritten. `frontend/public/sitemap.xml` and `llms.txt` cleaned.

### New schema (registered in `_SCHEMA` constant in `storage/_db.py`)

```sql
CREATE TABLE waitlist (
    email TEXT PRIMARY KEY, source TEXT NOT NULL DEFAULT '',
    created_at TEXT NOT NULL DEFAULT (datetime('now'))
);
CREATE TABLE github_stats_cache (
    key TEXT PRIMARY KEY, payload TEXT NOT NULL, fetched_at REAL NOT NULL
);
```

### New endpoints (registered inline in `web/app.py` before the SPA catch-all)

- `POST /api/v1/waitlist` — `{ email, source? }`. Pydantic `Literal["home_inline","hosted_page"]` for source. Helper layer in `storage/waitlist.py`.
- `GET /api/v1/github-stats` — returns `{ stars, contributors, last_commit_iso, fetched_at }`. Cache: L1 in-process memory + L2 SQLite, both 24h TTL, `asyncio.Lock` single-flight. Implementation: `web/github_stats.py`. Uses public wrappers `get_github_token()` and `github_get_with_headers()` in `tools/github_api.py` (the existing `_github_get` keeps its raise-on-error contract; new public wrapper swallows failures and returns `(None, headers)`).
- **No app-layer rate limit** on `/waitlist`; nginx `limit_req zone=waitlist rate=5r/m burst=3` is the gate (must be configured at deploy time — see Phase 2 follow-ups).

### New marketing components / pages

- `frontend/src/components/marketing/BuiltInOpen.tsx` — GitHub stats card. **Strict null-fallback**: any one of `stars`/`contributors`/`last_commit_iso` is null → render fallback link only.
- `frontend/src/components/marketing/HostedWaitlist.tsx` — email form, two variants (`inline` for home page, `page` for `/hosted`). variant ↔ source mapping enforced.
- `frontend/src/pages/ServicesPage.tsx` — Phase 1 skeleton (hero + CTA + placeholder note). Phase 2 will expand to 7-section IA per `new-positioning.md` § 1.2.
- `frontend/src/pages/HostedWaitlistPage.tsx` — wraps `HostedWaitlist` with `variant="page"`.

### 301 redirect map (`web/app.py`, before catch-all line ~1380)

```
/b2b-leads      → /services       (locale-preserved: /en/services, /zh/services)
/sample-data    → /services
/data-policy    → /                (locale-preserved: /en, /zh)
/seo-geo        → /services
```

12 routes total (4 paths × 3 locales). Both GET and HEAD methods. Factory function `_make_redirect(target)` avoids closure late-binding. Tests in `tests/test_redirects.py`.

### Phase 2 follow-ups (not yet done)

- Configure nginx `limit_req` for `/api/v1/waitlist` on newyork (snippet documented in `implementation-plan.md` §D.1.5; not yet applied to live nginx config).
- Manual batch email when hosted version ships: `SELECT email FROM waitlist;`. **Before sending**: configure SPF / DKIM / DMARC on aidcmo.com (`implementation-plan.md` §F.4 checklist).
- Translate ja/ko/es properly (currently EN-fallback for all repositioning copy).
- Expand `/services` from skeleton to full 7-section IA.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lling0000/OpenCMO](https://github.com/Lling0000/OpenCMO) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
