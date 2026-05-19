---
trigger: always_on
description: <!-- BEGIN:nextjs-agent-rules -->
---

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes -- APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

## Python Pipeline Workflow

Current priority is productionizing the morning enrichment pipeline in `scripts/`.

Order of work:
1. Fix entity parsing and non-human owner handling.
2. Add contact lookup dedupe/cache so repeated principals do not burn PDL credits.
3. Make SC SOS LLC resolution more deterministic before company-level PDL fallback.
4. Improve transaction/mortgage property-address extraction quality.
5. Only address evidence URL quality after output data is trustworthy.

Current status:
- Step 1 completed: non-human owners like "City of Greenville" no longer become fake people such as "Of City".
- Step 2 completed: contact enrichment now reuses existing `enriched_leads` contact info by normalized principal/entity, caches newly found contacts within the run, and caches negative no-match / no-contact person and company lookup results so repeated principals/entities do not keep hitting Apollo/PDL in the same run.
- Step 3 completed: SC SOS detail pages are now scored against the target LLC name before principal extraction, near-match entity variants are rejected, and only accepted SOS matches are allowed to short-circuit before company-level PDL fallback.
- Step 4 completed: property-address extraction now uses shared scored address parsing across GovOS detail HTML and mortgage OCR text, prefers labeled/property-context candidates, and rejects footer/party metadata false positives.
- Step 5 completed: evidence URLs are now normalized and ranked so enriched leads prefer concrete detail/article/document pages, and low-signal placeholders like search-query strings or login/disclaimer pages are no longer saved.
- Principal-quality validation completed: `principal_name` values are now screened before person-level contact enrichment so obvious junk labels, malformed token-order values, address-like strings, digit-heavy strings, and non-human entities are suppressed before Apollo/PDL person lookup.
- Historical repair/backfill pass completed: `scripts/enrich.py` now supports `--repair-stale` to re-run the current enrichment chain plus current contact gating/cache logic against stale `enriched_leads` rows, prioritizing legacy rows with no `enrichment_version`, rows with older `enrichment_version` values, and rows whose stored `principal_name` now fails current quality checks.
- Historical repair/backfill validation completed: dry-run validation against Supabase confirmed the stale-row selector was too chronological, so `--repair-stale` now scans a wider window and ranks obvious junk principals / stale pending rows ahead of generic legacy rows.
- Windows CLI compatibility completed: repair runs no longer die on `cp1252` console encoding because runtime status output in the enrichment scripts was normalized to ASCII.
- GIS owner normalization fix completed: GIS owner rows that still match the target entity are now preserved as non-human entities instead of being normalized into fake people such as `"Crafted Carolina"` or `"Ten Focus"` during stale repair.
- Mortgage credential loading fix completed: `scripts/enrich.py` now loads `.env.local` before importing `enrich_mort`, and `scripts/enrich_mort.py` now resolves `ROD_VIEWER_USERNAME` / `ROD_PASSWORD` at lookup time instead of freezing env vars at module import.
- Mortgage principal-quality gate completed: OCR-parsed signer labels such as `"Property Description"` and `"Legal Description"` are now rejected via shared `principal_name_quality_issue()` validation before mortgage results can short-circuit enrichment or mark a row enriched.
- Enrichment version bumped to `5`: `EnrichmentResult.is_enriched()` now requires a principal candidate to pass the shared quality gate, not just fail the non-human-owner check.
- Stale-repair selector fix completed: pending rows that legitimately retain a non-human entity principal are no longer re-queued forever just because `principal_name_quality_issue()` returns `non-human owner/entity`.
- Live repair validation completed: one row was successfully rewritten to version `5` in Supabase (`FOCUS TEN INVESTMENTS LLC`), and the stored row now remains `pending` with entity-level principal data instead of promoting junk mortgage OCR output to a fake human.
- Runtime note: full `python scripts/enrich.py --repair-stale` runs can take well over 20 minutes because each row may perform Playwright mortgage/GIS work plus multiple DuckDuckGo requests that can individually hit network timeouts.

Next step:
- Add a batch-size / limit control to `scripts/enrich.py --repair-stale` so live backfills can run in smaller chunks, then resume the live historical repair/backfill pass against Supabase and inspect the first chunked repaired rows.

Next productionization priorities:
1. Add a batch-size / limit control to `scripts/enrich.py --repair-stale`, then resume the live historical repair/backfill pass against Supabase and inspect the first chunked repaired rows.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jsteryous/rebbadvisors-website](https://github.com/jsteryous/rebbadvisors-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
