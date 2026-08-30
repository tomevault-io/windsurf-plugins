---
trigger: always_on
description: The guide for any coding agent working in this repository. `CLAUDE.md` points here; keep this file
---

# AGENTS.md

The guide for any coding agent working in this repository. `CLAUDE.md` points here; keep this file
as the single copy so the two cannot drift.

## Project Overview

Fredy is a self-hosted real estate finder for Germany. It scrapes German real estate portals (ImmoScout24, Immowelt, Immonet, Kleinanzeigen, WG-Gesucht, etc.), deduplicates results across providers, and sends notifications via Slack, Telegram, Email, Discord, ntfy, etc. It includes a React web UI and a built-in MCP server for LLM access to listings data.

- Node.js >= 22, ESM-only (`"type": "module"`)
- Default port: 9998, default login: admin / admin
- SQLite via `better-sqlite3` (synchronous - all DB ops are sync; only network I/O is async)

## Commands

```bash
# Development
yarn run start:backend:dev    # nodemon backend
yarn run start:frontend:dev   # Vite dev server (proxies /api → :9998)

# Production
yarn run start:backend        # NODE_ENV=production node index.js
yarn run build:frontend       # vite build → ui/public/

# Tests
yarn test                     # Live tests (hits actual providers)
yarn test:offline             # Offline tests using HTML/JSON fixtures (fast, preferred)
yarn test:download-fixtures   # Re-download fresh provider HTML fixtures

# Single test file
TEST_MODE=offline npx vitest run test/provider/immoscout.test.js

# Lint / Format
yarn lint && yarn lint:fix
yarn format && yarn format:check

# DB migrations
yarn migratedb
```

## Architecture

### Core data flow

```
index.js (startup)
  ├── runMigrations()
  ├── getProviders()            # lazily imports lib/provider/*.js
  ├── similarityCache.init()    # preloads hash cache from DB
  ├── api.js                    # starts fastify HTTP server
  └── initJobExecutionService() # registers event-bus listeners + starts scheduler

scheduler (every N minutes) or manual trigger via POST /api/jobs/:id/run
  └── FredyPipelineExecutioner.execute()
      1. queryStringMutator(url)           # inject sort-by-date param
      2. provider.getListings()            # API or Puppeteer+Cheerio
      3. provider.normalize(listing)       # raw → ParsedListing
      4. provider.filter(listing)          # blacklist + required fields
      5. filter to hashes not yet in DB
      6. provider.fetchDetails()           # optional enrichment
      7. geocodeAddress()                  # optional lat/lng
      8. storeListings()
      9. similarityCache.checkAndAddEntry() # cross-provider dedup (exact hash, then fingerprint)
      10. _filterBySpecs() + _filterByArea()
      11. notify.send()                    # fan-out to all adapters
```

### Plugin systems

**Providers** (`lib/provider/*.js`) - each module exports:
- `metaInformation` - `{ id, name, baseUrl }`, plus an optional `countries` (ISO 3166-1 alpha-2,
  lowercase). Absent means `['de']`, which is why no shipped provider declares it and why adding the
  field changed no existing installation. Resolved in `lib/services/providers/`: `countries.js` is
  the pure half (the default, normalisation, union) and is all the Nominatim client imports, since
  `providerCountries.js` reaches for the job storage and would drag SQLite in behind it. The
  geocoder searches within the resolved countries; the map's `maxBounds` is the union of their
  boxes from `ui/src/components/map/countryBounds.js`. Where no provider exists to ask - home
  addresses, the listings map, the listing detail - the answer is the union across the jobs the user
  can see, and where the job form is open it is the providers ticked in it
- `config` - the **static** `ProviderConfig` template: `requiredFieldNames`, `crawlContainer`, `crawlFields`, `sortByDateParam`, `normalize()`, optional `getListings()`, `fetchDetails()`, `activeTester()`. `url` is `null` here and there is no bound `filter`.
- `createConfig(sourceConfig, blacklist)` - returns a **fresh** `ProviderConfig` per job run: the template plus this run's `url`, `enabled`, and a `filter` closed over this run's blacklist.

Providers are **stateless**. Nothing run-specific may live at module scope: two jobs can execute
concurrently (a manual run started while the scheduler is working), and shared mutable state let
the second job overwrite the first one's URL and blacklist mid-run, storing listings under the
wrong job. The same rule is why the Cheerio parser builds its document inside `parse()` instead of
keeping a module-level `$`.

**Notification adapters** (`lib/notification/adapter/*.js`) - each exports:
- `config` - `{ id, name, description, fields }` (drives the UI form)
- `send({ serviceName, newListings, notificationConfig, jobKey, baseUrl })`
- Loaded dynamically at startup via `fs.readdirSync`

Field definitions carry two optional flags that the UI and the API read declaratively, so neither
needs per-adapter code:
- `secret: true` - a credential. Never serialised to anyone who may not edit the channel, and
  masked in the form. Every token, password, API key and webhook URL must carry it.
- `target: true` - the one field naming the destination. Drives the "Destination" column.

An adapter *configuration* is separate from the adapter itself: it is a row in `configured_adapter`
("a notification channel" in the UI) that many jobs can reference.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [orangecoding/fredy](https://github.com/orangecoding/fredy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
