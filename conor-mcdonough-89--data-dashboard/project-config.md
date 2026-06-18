---
trigger: always_on
description: Password-gated, multi-sport seller analytics dashboard for SidelineSwap. For each sport (tennis, hockey, baseball, softball, golf, lacrosse) it shows the **top 100 sellers by 8-week GMV** with a weekly heat-map table, a US/Canada seller map, per-seller GMV trend chart, and five seller-insight tabs (Trajectory, AOV Leaders, Consistency, Specialists, Subscriptions). Data comes from either CSV upload or a direct **Metabase Sync** that runs the canonical SQL queries against the warehouse.
---

# SidelineSwap Seller Dashboard

Password-gated, multi-sport seller analytics dashboard for SidelineSwap. For each sport (tennis, hockey, baseball, softball, golf, lacrosse) it shows the **top 100 sellers by 8-week GMV** with a weekly heat-map table, a US/Canada seller map, per-seller GMV trend chart, and five seller-insight tabs (Trajectory, AOV Leaders, Consistency, Specialists, Subscriptions). Data comes from either CSV upload or a direct **Metabase Sync** that runs the canonical SQL queries against the warehouse.

## Tech stack

Vanilla HTML/CSS/JS, no build step, no framework. Libraries loaded from CDN: **Leaflet 1.9.4** (map), **Chart.js 4.4.0** (trends), **PapaParse 5.4.1** (CSV). **IndexedDB** persists per-sport data and a geocoder cache. Hosted on **Vercel** with one Edge Function for the Metabase proxy. A zero-dependency Node `server.js` is included as a self-host fallback.

## File map

| File | Role |
| --- | --- |
| `index.html` | Single-page shell. Contains the importer overlay, sync overlay, password gate, confirm dialog. Loads scripts in order: geocoder, insights, importer, metabase, dashboard. |
| `dashboard.js` | All UI rendering. `Dashboard.init()` is the entry point. Contains the `SyncUI` helper module that drives the Metabase sync overlay. |
| `importer.js` | CSV parsing + the shared post-parse pipeline `importSportFromArrays(name, sellers, swaps, onProgress)` used by both CSV and Metabase paths. Also handles JSON snapshot export/import. |
| `metabase.js` | Metabase config, six-category whitelist, two SQL templates, session-token auth (`POST /api/session`) with API-key fallback, `syncCategory` / `syncAll`. All HTTP goes through `/api/metabase/*`. |
| `insights.js` | Pure compute. `Insights.compute(sellers, swaps, numWeeks)` → per-seller insight objects keyed by seller_id. `Insights.getSpecialistCategories(swaps)` → top 10 `item_category_2` values for the Specialists tab. |
| `geocoder.js` | Owns the IndexedDB schema (object stores `sports` and `geo-cache`). Public API: `openDB`, `detectFormat`, `loadBundledData`, `resolve`, `resolveBatch`, `GEO_DATA_VERSION`. Resolves zips via memory cache → IndexedDB → bundled CSVs → Nominatim (rate-limited, 1 req/sec). |
| `data/us_zips.csv`, `data/ca_fsa.csv` | Bundled lat/lng tables (2024 Census Gazetteer for US, FSA centroids for CA). `GEO_DATA_VERSION = 2` — bumping it triggers a refresh of stored sport coordinates on next load. |
| `style.css` | All styles, hand-written, dark theme. Variables defined in `:root`. |
| `api/metabase-proxy.js` | Vercel **Edge function**. Streams every `/api/metabase/*` request to `${METABASE_URL}/*`. Auto-prepends `https://` if scheme missing, validates the URL upfront, includes the resolved target in upstream-error responses. |
| `vercel.json` | Single rewrite: `/api/metabase/:path*` → `/api/metabase-proxy?p=:path*`. |
| `server.js` | Zero-dependency Node fallback for self-hosting (serves static files + same proxy). Not used on Vercel. |
| `package.json` | No deps. `npm start` runs `server.js`. Requires Node ≥18. |

## Data model

### `sportData` (one row per sport in the `sports` IndexedDB store)

```
{
  id:              "hockey",                  // slug derived from name
  name:            "Hockey",                  // display name
  detectedSport:   "hockey",                  // item_category_1 value
  sellers:         [ <seller row>, ... ],     // ≤100 rows
  numWeeks:        8,
  insights:        { <seller_id>: <insight>, ... },
  specialistCats:  [ "ice-hockey-skates", ... ],
  zipCoords:       { "02115": { lat, lng }, "M5V": { lat, lng }, ... },
  mapConfig:       { center: [lat, lng], zoom: N },
  geoDataVersion:  2,
  importedAt:      "2024-…"
}
```

### Seller row (input from query 1 / GMV CSV)

`seller_id`, `gmv_rank`, `total_gmv`, `week_1`…`week_N`, `ship_from_state`, `ship_from_zip`, `trade_in_account` (bool), `power_seller` (bool), `pro_seller` (bool), `pro_plus_seller` (bool), `category` (= `item_category_1`).

### Swap row (input from query 2 / Swaps CSV)

`paid_at`, `buyer_join_date`, `buyer_city`, `buyer_state`, `buyer_purchase_date`, `seller_id`, `cash_offer_amount`, `item_category_2`, `item_condition`. The repeat-buyer heuristic in `insights.js` compares `buyer_purchase_date` to `paid_at` — if the buyer's first-purchase date predates this swap's `paid_at`, it's a returning buyer.

## Key flows

**CSV import.** User clicks `+` tab → drops two CSVs → `parseCSV` (PapaParse) → `importSportFromArrays(name, sellers, swaps)` → `Insights.compute` → `Geocoder.loadBundledData` + `Geocoder.resolveBatch` → `saveSport` to IndexedDB → `Dashboard.onSportImported` re-renders.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [conor-mcdonough-89/data-dashboard](https://github.com/conor-mcdonough-89/data-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
