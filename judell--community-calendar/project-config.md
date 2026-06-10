---
trigger: always_on
description: Required: at the start of every session, and before analysis or edits, read
---

# Agent Strategies for Calendar Source Discovery

## Required External Guidance

Required: at the start of every session, and before analysis or edits, read
`.claude/xmlui-desktop-conventions.md`. Treat it as binding repo guidance
for all work in this repository.

## Table of Contents

- [Discovery Philosophy](#discovery-philosophy)
- [Quick Reference: Adding a New City](#quick-reference-adding-a-new-city)
- [App Architecture](#app-architecture)
  - [SOURCES_CHECKLIST.md](#sources_checklistmd)
- [Editing the Workflow YAML](#editing-the-workflow-yaml)
- [Quick Reference: Adding a New Scraper](#quick-reference-adding-a-new-scraper)
  - [Verification Checklist](#verification-checklist)
  - [Recommended: Use the add_scraper script](#recommended-use-the-add_scraper-script)
  - [feeds.txt is auto-generated](#feedstxt-is-auto-generated)
- [Quick Reference: Adding a New ICS Feed](#quick-reference-adding-a-new-ics-feed)
- [Reusable Scrapers](#reusable-scrapers)
  - [MaxPreps](#maxpreps-scrapersmaxprepspy---high-school-athletics)
  - [GrowthZone](#growthzone-scrapersgrowthzonepy---chamber-of-commerce)
  - [Library Intercept](#library-intercept-scriptslibrary_interceptpy)
  - [Elfsight Calendar](#elfsight-calendar-scraperslibelfsigtpy)
  - [Legistar](#legistar-scraperslegistarpy---city-government-meetings)
  - [Bibliocommons](#bibliocommons-scraperslibbibliocommunspy---library-event-platforms)
  - [GoDaddy Calendar](#godaddy-calendar-scraperslibgodaddypy---godaddy-website-builder)
  - [Mobilize.us](#mobilizeus-scrapersmobilizepy---civic-and-political-organizing)
- [Platform-Specific Techniques](#platform-specific-techniques)
  - [Drupal](#drupal)
  - [SeeTickets Widgets](#seetickets-widgets)
  - [Wix Events](#wix-events)
- [Source Discovery](#source-discovery)
  - [Strategy 1: Ticketing Platform Indirection](#strategy-1-ticketing-platform-indirection)
  - [Strategy 2: Topical Search for New Cities](#strategy-2-topical-search-for-new-cities)
  - [Strategy 3: Meetup ICS Pattern](#strategy-3-meetup-ics-pattern)
  - [Strategy 4: Platform-Specific Discovery](#strategy-4-platform-specific-discovery)
- [Pipeline Validation](#pipeline-validation)
  - [Validation Script](#validation-script)
  - [Common Silent Failure Causes](#common-silent-failure-causes)
- [Known Platform Limitations](#known-platform-limitations)

## Discovery Philosophy

**We want COMPLETE coverage, not curated coverage.** This means:

1. **Long-tail events matter** - A book club with 5 members, a small craft meetup, a neighborhood cleanup - these ARE our target. Don't skip sources just because they seem niche or low-volume.

2. **Schools are gold mines** - High schools and colleges have athletic events, theater, band concerts, art shows, parent nights, graduation ceremonies.

3. **Churches and community centers** - Weekly services may not be interesting, but special events (concerts, fundraisers, community dinners) are.

4. **Youth sports leagues** - Worth checking but rarely viable; most use member-only platforms (see [discovery lessons](docs/discovery-lessons.md)).

5. **If in doubt, add it** - We can always filter later. Missing events is worse than having too many.

## Quick Reference: Adding a New City

1. **Create city directory** under `cities/` with `SOURCES_CHECKLIST.md`
2. **Run source discovery** — platform searches (Tockify, WordPress `?ical=1`, Meetup ICS), topical searches. See [docs/curator-guide.md](docs/curator-guide.md). Run the playbook first, assess gaps second.
3. **Update the GitHub Actions workflow** (`.github/workflows/generate-calendar.yml`):
   - Add city to the locations list (line with `echo "list=..."`)
   - Add a city section with curl commands for all feeds + `combine_ics.py` call
   - Add city to the backup/restore lists in the commit step (`for city in ...`)
4. **Source metadata** — for feeds, the Supabase `feeds` table is the source of truth in the main repo. `feeds.txt` is exported from the DB during builds. For scrapers, `add_scraper.py` adds the workflow invocation and a generated `feeds.txt` entry/metadata path is preserved via the export step. No manual dict editing needed.
5. **Add city to UI** — TWO places must be updated:
   - `index.html`: add entry to `cityNames` map (e.g., `toronto: 'Toronto'`)
   - `Main.xmlui`: add a Button in the city picker VStack (search for "Choose your city")
6. **Add city to load-events function** — add URL entry to `EVENTS_URLS` in `supabase/functions/load-events/index.ts`, then redeploy the edge function
7. **Optionally set up geo-filtering** — create `city.conf` if feeds include events outside your area. This is optional; if the file doesn't exist, all events pass through. See [docs/procedures.md](docs/procedures.md#5-geo-filtering-setup).
8. **Update SOURCES_CHECKLIST.md** — document findings, track pending sources
9. **Commit and push** — workflow runs daily or trigger manually

## App Architecture

The XMLUI app lives at the repo root and serves all cities from https://judell.github.io/community-calendar/.

- **`index.html`** — Entry point. Reads `?city=` URL param to set `window.cityFilter` and `window.cityName`. No param shows the city picker.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [judell/community-calendar](https://github.com/judell/community-calendar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
