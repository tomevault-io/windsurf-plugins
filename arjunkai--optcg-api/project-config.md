---
trigger: always_on
description: REST API serving One Piece TCG card + price data, backing OPBindr and public consumers.
---

# OPTCG API

REST API serving One Piece TCG card + price data, backing OPBindr and public consumers.

## Stack
- **Runtime:** Cloudflare Workers (JavaScript, Hono router)
- **Database:** Cloudflare D1 (SQLite)
- **Scrapers:** Python + Playwright (official site for cards, TCGPlayer price guides for pricing)

Live: `https://optcg-api.arjunbansal-ai.workers.dev`  ·  Docs: `/docs`  ·  OpenAPI: `/openapi.json`

## Critical Rules
- Never hand-edit D1 schema — always add a numbered migration under `migrations/` and re-run `npx wrangler d1 execute optcg-cards --remote --file=migrations/NNN.sql`
- `schema.sql` mirrors a fresh-create state; keep it in sync with each migration
- TCGPlayer is the authoritative price source; don't swap to third-party APIs (dotgg.gg etc.) without a memory-backed reason
- CORS headers live in `src/index.js` — don't strip them from new routes

## Key files
- `src/index.js` — Hono router + CORS, registers all routes
- `src/cards.js` — `/cards` search + `/cards/:id` single-card endpoints
- `src/sets.js` — `/sets` list + `/sets/:id/cards` set-cards
- `src/images.js` — `/images/:card_id` proxy. Order: R2 (`cards/{id}.png`) → DON TCGPlayer CDN via D1 `tcg_ids` → official site fallback for regular cards
- `src/docs.js` — OpenAPI spec + Scalar docs page
- `src/db.js` — row → JSON normalization (handles JSON-encoded columns)
- `schema.sql` — full schema snapshot
- `migrations/` — numbered D1 migrations
- `scraper.py` — official site card data scrape → `data/cards.json`
- `classify_variants.py` — manga/serial classification via Limitless TCG
- `scripts/scrape_tcgplayer_prices_pw.py` — Playwright price scrape (primary, no credits)
- `scripts/scrape_tcgplayer_prices.py` — Firecrawl fallback
- `scripts/build_all_prices.py` — parse + map prices → `data/card_prices_all.json`
- `scripts/build_don_cards.py` — deduped DON catalog → `data/don_cards.json`
- `scripts/import-d1.js` / `import-prices-d1.js` / `import-don-d1.js` — batched D1 writes
- `scripts/import-jp-exclusives.js` — seeds JP-exclusive Championship variants from `data/jp_exclusives.json` into `cards` + `card_sets`, inheriting stats from the base row
- `scripts/price_jp_exclusives.py` — eBay Browse API pricing for the JP exclusives, uses each entry's `note` or `image_search_query` as the search and stamps `price_source='ebay_jp'`
- `scripts/fetch_card_image.py` — eBay-sourced card images for cards Bandai doesn't publish cleanly (JP exclusives, DON cards). Adaptive card-bounds detection, aspect-aware scoring (`card_area × sharpness × card_fill × aspect_bonus`), blocklist of slabbed/sealed listings, and a `--min-card-px` floor so it never downgrades an existing image. Uploads to R2 at `optcg-images/cards/{id}.png` then calls `/cards/all?refresh=1` to purge the edge cache. Flags: `--all` (all JP exclusives), `--all-dons` (all DON rows from D1), `<card_id>` (single card with D1 fallback), `--dry-run`, `--force`.
- `.github/workflows/scrape.yml` — weekly auto-refresh of everything
- `scripts/ptcg-fetch.js` / `scripts/ptcg-import-d1.js` — Pokémon TCG bulk import. Fetch caches TCGdex API responses to `data/ptcg_cache/{sets,cards}-{lang}.json`; import generates batched SQL in `scripts/ptcg_batches/` and runs them via `wrangler d1 execute --remote`. Resumable (re-running fetch only fills missing cards). See "Pokémon TCG import" below.

## Pokémon TCG import

Bulk-loads card + set data for all four languages (`en`, `ja`, `zh-cn`, `zh-tw`) into `ptcg_cards` and `ptcg_sets`. Source: TCGdex public API (https://api.tcgdex.net).

**Scope:** ~22k EN cards plus several thousand each in ja/zh-cn/zh-tw — total approaching 88k. Fetch wallclock at concurrency=8 is roughly 30–60 minutes per language. Import (D1 writes) runs in a few minutes once cached.

**Why a Node script vs. a Worker Cron Trigger:** Workers cap at ~15-min wallclock; the initial bulk fetch is hours. The Cron Trigger pattern documented in the multi-game plan is the right shape for the *daily delta* once initial data lands — not for the initial seed.

**Steps:**
```
node scripts/ptcg-fetch.js                          # all 4 langs, ~hours
node scripts/ptcg-import-d1.js                      # all cached langs → D1
```

**Test/partial runs:**
```
node scripts/ptcg-fetch.js --lang=en --set=base1    # 102 cards, seconds
node scripts/ptcg-import-d1.js --lang=en --dry-run  # write SQL, skip D1
```

**Resume:** re-running `ptcg-fetch.js` skips cards already in `data/ptcg_cache/cards-{lang}.json`. The fetch flushes to disk every 200 cards so a crash doesn't lose progress.

**Verify after import:**
```
npx wrangler d1 execute optcg-cards --remote --command "select lang, count(*) from ptcg_cards group by lang"
```

## Refresh pipeline

Automated: runs every Monday 6am UTC via GitHub Actions. Manual trigger: `gh workflow run "Weekly Card Scrape"`.

Local full refresh:
```
python scraper.py                                  # cards
python classify_variants.py                        # variant types
node scripts/import-d1.js                          # -> D1
.venv/Scripts/python.exe scripts/scrape_tcgplayer_prices_pw.py
python scripts/build_all_prices.py
node scripts/import-prices-d1.js                   # -> D1
python scripts/build_don_cards.py

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arjunkai/optcg-api](https://github.com/arjunkai/optcg-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
