---
trigger: always_on
description: Flask web app for researching Indian allotment land dispossession. Replaces a legacy PHP search at land-sales.iath.virginia.edu. Built at IATH, University of Virginia.
---

# Federal Register Forced Fee App — Development Guide

## What This Is
Flask web app for researching Indian allotment land dispossession. Replaces a legacy PHP search at land-sales.iath.virginia.edu. Built at IATH, University of Virginia.

## Running
```bash
cd federal-register-app
source venv/bin/activate
python3 app.py  # runs on http://localhost:5001
```

## Stack
- **Backend:** Flask + psycopg2, Python 3
- **Database:** PostgreSQL `allotment_research` (local, user=cwm6W; Cloud SQL in production)
- **Frontend:** Bootstrap 5, jQuery, DataTables (server-side), Chart.js
- **Map:** Leaflet.js, leaflet.heat, Esri ArcGIS Feature Service (standalone SPA at `/map`)
- **Deployment:** Google Cloud Run, **manual** (there is NO Cloud Build trigger — pushing to
  main does NOT auto-deploy, despite older notes). To deploy: build from a CLEAN context (the
  working tree has uncommitted junk) and update the service. Gotchas verified 2026-06-24:
  (1) `gcloud builds submit` falls back to `.gitignore` without a `.gcloudignore`, and `.gitignore`
  has `data/` — which silently drops `static/data/us-states.geojson` from the image; add a
  permissive `.gcloudignore`. (2) The Cloud SQL data and the code deploy are independent — push
  table changes via `scripts/push_new_tables_to_cloudsql.sh` (Auth Proxy on a non-5432 port),
  deploy code separately. Build+deploy used:
  `git archive HEAD | tar -x -C /tmp/ctx; printf '.git/\n' > /tmp/ctx/.gcloudignore;
   gcloud builds submit /tmp/ctx --tag us-east1-docker.pkg.dev/$PID/cloud-run-source-deploy/federal-register-app:TAG;
   gcloud run deploy federal-register-app --region us-east1 --image …:TAG` (image-only deploy
  retains the existing DATABASE_URL env var and Cloud SQL connection).
- **Virtualenv:** `./venv/`

## Architecture
Single-file Flask app (`app.py`) with Jinja2 templates. No ORM — raw SQL with psycopg2. All tables use server-side DataTables pagination via JSON API endpoints.

### Two parallel sections

**Claims section** (original) — 35,686 Federal Register claims from two 1983 publications ([March 31](https://land-sales.iath.virginia.edu/documents/federal_register/fedreg-1983_03_31.pdf) and [November 7](https://land-sales.iath.virginia.edu/documents/federal_register/fedreg-1983_11_07.pdf)):
- `/` — Claims search (DataTables + filters). Columns: BIA Agency Code, Case #, Allottee Name, Tribe, Allotment #, Claim Type, Patent Date, Map (yes/no badge). Default sort: agency code, then case number.
- `/claim/<id>` — Claim detail with linked patents. Document Source links to original FR PDF.
- `/api/search` — JSON API for claims DataTables
- `/api/search/csv` — CSV download

**Patents section** (added March 2026) — 285,870 BLM allotment patents:
- `/patents` — Patent search (DataTables + filters for name/tribe/state/type/date)
- `/patent/<objectid>` — Patent detail with PLSS land description (+ "Patent Cancelled" block for records in cancelled_patent_research)
- `/api/patents` — JSON API for patents DataTables (filters incl. `cancelled=yes`)
- `/api/patents/csv` — CSV download
- `/patents/cancelled` — Browse the cancelled-patents research dataset (cancelled_patent_research, 439 records); summary by cancellation authority + full list. Filter constant: `CANCELLED_RESEARCH_WHERE_SQL`
- `/patents/timeline` — Stacked bar chart (fee vs trust, forced fee toggle)
- `/api/patents/timeline` — JSON API for timeline

**Map (integrated Leaflet SPA):**
- `/map` — Interactive allotment patent map (Leaflet + Esri Feature Service, standalone template)

**Other pages:**
- `/tribes` — Tribe list with claim counts and BIA agency codes
- `/tribe/<slug>` — Individual tribe page with timeline, agency codes in header
- `/timeline` — Forced fee claims timeline (original)
- `/about` — About page (includes BIA agency code reference table, FR PDF links)
- `/research` — Research overview (home.html) with dataset cards and FR PDF links

### Cross-links
- Claim detail → BLM patent: "View full BLM record" link (via accession_number lookup in blm_allotment_patents)
- Patent detail → Claim: alert banner linking to Federal Register claim (via forced_fee_patents_rails)

## Key Database Tables
See `DATABASE.md` for full schemas.

- `federal_register_claims` (35,686 rows) — FR claims (all types). PK: id
- `forced_fee_patents_rails` (17,560 rows) — hand-verified claim-to-patent linkages from Rails admin
- `rails_patents` (285,870 rows) — full patent catalog with `has_plss_geometry` flag. PK: id
- `blm_allotment_patents` (239,845 rows) — BLM patent mirror from ArcGIS (mappable patents only). PK: objectid
- `all_patents` (view, 285,870 rows) — unified view joining rails_patents + blm_allotment_patents
- `fee_patents` (88,537) / `trust_patents` (95,353) — older BLM patent tables (still used for claim detail fallback)
- `trust_fee_linkages` (29,229) — trust→fee conversion records
- `parcels_patents_by_tribe` (401,811) — PLSS legal descriptions

### Claims → Patents join
```sql
LTRIM(fr.case_number, '0') = LTRIM(ffp.case_number, '0')
AND fr.allottee_name = ffp.fedreg_allottee
```

### Patent authority categories (defined in app.py)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cwmmwc/american-indian-allotment](https://github.com/cwmmwc/american-indian-allotment) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
