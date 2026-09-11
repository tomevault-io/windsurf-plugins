---
trigger: always_on
description: Guide for AI agents and contributors working in this repository.
---

# AGENTS.md — BidetBud

Guide for AI agents and contributors working in this repository.

## Project summary

**BidetBud** is a static, client-only map for finding bidet-equipped restrooms — masajid, restaurants, hotels, and public spots — in the **USA, UK, Canada, and Singapore**.

- **Live:** [bidetbud.com](https://bidetbud.com/)
- **Stack:** Single HTML page + CSS. No build step, no backend, no framework.
- **Map:** Leaflet + MarkerCluster (bundled inline in `index.html`)
- **Data:** `bidet-seed.json` fetched async at boot (`data/bidet-restaurants.json` is the full source of truth)
- **Submissions:** In-page form via Web3Forms (`WEB3FORMS_ACCESS_KEY` in `js/app.js`; public client key)
- **Analytics:** GoatCounter (`bidetbud.goatcounter.com`)

Related but separate project: [bidetbud.com](https://www.bidetbud.com/) is the Singapore-only PWA (“Bidet Bud SG”). Singapore data for this repo is imported from its public JSON export.

---

## Repository layout

```
index.html                          App shell (~22 KB); logic in js/app.js; seed in bidet-seed.json
css/app.css                         All UI styles (Inter font, zinc palette)
css/github-star.css                 Footer link styles
images/                             Logo and favicons
data/singapore-bidets.geolocation.json   Cached SG source (community bidet sightings only)
data/france-verified-bidets.json         Curated FR rows with cited evidence (not bulk OSM)
data/russia-verified-bidets.json         Curated RU rows with cited evidence (Russian sources preferred)
data/russia-scrape-candidates.json       Auto-scrape hits from scripts/scrape-russia-sources.cjs (review only)
scripts/
  apply-address-fixes.cjs           Re-geocode seed; manual coordinate overrides
  import-singapore.cjs              Merge @toiletswithbidetsg / Bidet Bud SG JSON
  import-france.cjs                 Merge only rows from france-verified-bidets.json
  import-russia.cjs                 Replace all Russia rows from russia-verified-bidets.json
  scrape-russia-sources.cjs         Crawl Russian booking/review sites for bidet mentions
  scrape-russia-exhaustive.cjs      90-min exhaustive crawl with URL discovery
  merge-russia-candidates.cjs      Bulk-merge 101hotels scrape hits into verified JSON
  geocode-russia.cjs                  Apply manual GPS overrides for Russia curated rows
  scrape-toto-references.cjs        Fetch all TOTO Europe WASHLET case studies
  finish-toto-references.cjs        Append manual coords for ambiguous TOTO venues
  import-toto-references.cjs        Merge TOTO references into BIDETBUD_SEED
  scrape-toto-try.cjs               Parse eu.toto.com "Try WASHLET" finder listing
  geocode-toto-try.cjs              Geocode Try-WASHLET rows (Photon/Nominatim, cached)
  geocode-toto-try-retry.cjs        Smarter retry for unresolved Try-WASHLET rows
  import-toto-try.cjs               Merge Try-WASHLET showrooms/dealers into BIDETBUD_SEED
  lib/toto-try.cjs                  Shared country inference + address helpers
  address-fix-report.json           Output from geocode script (optional)
```

---

## Architecture constraints

**Keep it static.** Do not add:

- Node/npm build pipelines, React/Vue, or SSR
- Service workers, PWA manifests, or install prompts (removed intentionally)
- Backend APIs or server-side geocoding at runtime

**Do use:**

- Vanilla JS in `js/app.js` (loaded deferred after Leaflet)
- Styles in `css/app.css` — avoid large new inline style blocks
- Node scripts in `scripts/` for one-off data maintenance only

---

## Verification policy (critical)

**Only add locations where a bidet is explicitly confirmed — never assume.**

| OK | Not OK |
|----|--------|
| User personally verified (`bidetStatus: "verified"`) | Bulk-importing all mosques from OSM “because wudu exists” |
| Community bidet-only lists (e.g. [@toiletswithbidetsg](https://www.instagram.com/toiletswithbidetsg/)) | Generic halal restaurant or public toilet directories |
| Manufacturer case studies naming WASHLET/bidet install | Places “likely” to have bidets based on country/culture |
| Reviews/articles that explicitly mention bidet, washlet, douchette, or handheld spray | Ablution/wudu alone without bidet/spray evidence |

Every `internet` or `warmed` entry **must** have `sourceUrl` + `sourceQuote` citing the evidence. Use `verifiedMethod` when helpful (`community-sighting`, `manufacturer-reference`, `web-source`).

---

## Seed data (`BIDETBUD_SEED`)

Each location is a JSON object. Only entries with `bidetStatus` of `verified`, `warmed`, or `internet` appear on the map.

### Required / common fields

| Field | Notes |
|-------|-------|
| `name` | Display name |
| `address` | Street address |
| `latitude`, `longitude` | Strings (e.g. `"37.8619778"`) |
| `city` | e.g. `"Berkeley, CA"` or `"East"` (SG region) |
| `country` | `USA`, `UK`, `Canada`, or `Singapore` |
| `type` | `mosque`, `restaurant`, `hotel`, or `public` |
| `bidetStatus` | `verified` (user), `warmed` (heated/TOTO), `internet` (web source) |
| `access` | `public` or `limited` |

### Optional fields

| Field | Notes |
|-------|-------|
| `bidetType` | e.g. `"Handheld sprayer"`, `"Heated seat"` |
| `sourceUrl` | Required for `internet` entries |
| `sourceQuote` | Short attribution |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ymorsi7/bidetbud](https://github.com/ymorsi7/bidetbud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
