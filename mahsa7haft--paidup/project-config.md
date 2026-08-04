---
trigger: always_on
description: Instructions for Claude Code. This file overrides default behaviour — follow it exactly.
---

# CLAUDE.md — PaidUp

Instructions for Claude Code. This file overrides default behaviour — follow it exactly.

---

## Run commands

```bash
# Install dependencies
uv sync

# Run locally
PYTHONPATH=src uv run python -m app.main
# → http://localhost:5002

# Run tests
uv run pytest

# Seed R2 card cache for all current MPs
PYTHONPATH=src uv run python -m app.seed_cards

# Seed donor tags from CSV
PYTHONPATH=src uv run python -m app.seed_tags data/tags.csv
```

---

## Architecture

### Request flow

```
Browser POST /lookup
  → cache.get("paidup:lookup:{name}")     return immediately if hit (Redis, 1h TTL)
  → search_mp()                            Parliament Members API → member_id, name, party
  → [parallel, ThreadPoolExecutor(3)]
      → get_interests() + parse_interests() + deduplicate_donors()
      → get_biography() + parse_biography()
      → get_twfy_data()
  → db.apply_donor_tags()                  attach tags from donor_tags table
  → cache.set(result, ttl=1h)
  → cache.set("paidup:interests:{id}", deduped_interests, ttl=1h)
  → return JSON

Browser POST /analyze  ← two-level cache
  → cache.get(...)                         Redis, 24h TTL
  → db.get_analysis(...)                   Postgres analyses table, 28-day TTL
  → analyze() in ai.py                     Claude claude-sonnet-4-6 — only on full miss
      → loads prompt from prompts/{key}_v{n}.txt
  → db.save_analysis(...)
  → cache.set(...)
  → return JSON

GET /card/{member_id}
  → r2.get_card_url(member_id)             R2 CDN hit → redirect (skip generation)
  → _get_deduped_interests(member_id)      Redis hit or fresh Parliament fetch
  → generate_card()                        Pillow 900×500 landscape PNG
  → r2.upload_card(member_id)
  → return PNG (or redirect to CDN URL)

GET /card/{member_id}/mobile
  → r2.get_card_url(member_id, variant="mobile")
  → _get_deduped_interests(member_id)
  → generate_mobile_card()                 Pillow 500×900 portrait PNG
  → r2.upload_card(member_id, variant="mobile")
  → return PNG (or redirect to CDN URL)

GET /card/{member_id}/badges
  → _get_deduped_interests(member_id)
  → get_badge_layout()                     JSON — badge positions for overlay (desktop only)

GET /metrics
  → PrometheusMetrics scrape endpoint      latency histograms, request counters, error rates
```

### Module responsibilities

| File | Responsibility |
|---|---|
| `main.py` | Flask routes only. Orchestrates calls to other modules; no business logic. |
| `parliament.py` | All UK Parliament API calls (Members + Interests). Also owns `deduplicate_donors`. |
| `theyworkforyou.py` | TheyWorkForYou API. Always returns `None` gracefully if key not set. |
| `ai.py` | Anthropic SDK calls. Prompts loaded from disk at call time. |
| `card.py` | Pillow image generation. Landscape (`generate_card`) and portrait mobile (`generate_mobile_card`). |
| `cache.py` | Redis wrapper (L1). Returns `None` / no-ops silently when Redis is unavailable. |
| `database.py` | PostgreSQL layer (L2). Stores AI analyses for 28 days. No-ops when DATABASE_URL is unset. |
| `r2.py` | Cloudflare R2 card image cache. Accepts `variant` param for desktop vs mobile keys. |
| `metrics.py` | Prometheus counter definitions. Import here to avoid circular imports. |
| `text_utils.py` | `normalize_name()` and `best_fuzzy_match()` — shared between parliament.py and database.py. |
| `seed_cards.py` | CLI script: generate + upload all MP cards to R2. Not imported. |
| `seed_tags.py` | CLI script: bulk-load donor_tags from CSV. Not imported. |

---

## Key design decisions

### Parallel /lookup API calls

`search_mp()` runs first (sequential — need `member_id`). Then `get_interests`,
`get_biography`, and `get_twfy_data` run in parallel via `ThreadPoolExecutor(max_workers=3)`.
Both `search_mp` (10s) and `get_interests` (15s) have explicit timeouts — Parliament's API
has been observed hanging indefinitely without them.

### _get_deduped_interests cache

`/card`, `/card/mobile`, and `/card/badges` all need the same deduped interest list.
`_get_deduped_interests(member_id)` checks `paidup:interests:{id}` in Redis first.
`/lookup` writes to this key after computing, so a card request after a search is a Redis hit
(~5ms) instead of a fresh Parliament API call + TF-IDF (~500–800ms).

If you add a new route that needs interests, use `_get_deduped_interests()` — not a bare
`get_interests()` call.

### Two-level cache for /analyze

`/analyze` uses L1 (Redis, 24h) → L2 (Postgres, 28 days) → Claude API.

- **L1 Redis**: fastest — did someone search this MP in the last 24h?
- **L2 Postgres**: persistent across restarts — have we ever analysed this MP this month?
- **Claude**: only reached when both miss

On L2 hit, Redis is repopulated so the next call doesn't touch Postgres.

The `_cached` field in the response shows which layer served it (`"redis"`, `"db"`, or absent).

### Why 28 days for Postgres TTL

Parliament's Register of Members' Financial Interests is updated within 28 days of changes.
After 28 days an analysis could reference stale data so it is discarded and regenerated.

### Prompt versioning

Prompts live in `src/app/prompts/` as `{key}_v{n}.txt`. `ai.py` picks the highest version

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mahsa7haft/paidup](https://github.com/mahsa7haft/paidup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
