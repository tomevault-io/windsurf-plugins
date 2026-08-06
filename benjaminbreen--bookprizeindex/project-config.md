---
trigger: always_on
description: The Book Prize Index is a static-first Next.js app for exploring major nonfiction book prize records. It is intended to be a free public resource for writers, researchers, publishers, librarians, and readers.
---

# Agent Notes

## Project Overview

The Book Prize Index is a static-first Next.js app for exploring major nonfiction book prize records. It is intended to be a free public resource for writers, researchers, publishers, librarians, and readers.

The current app is a polished prototype with an expanded but still incomplete corpus. The most important ongoing work is turning it into a trustworthy, source-backed nonfiction award index with reviewed book metadata, imprints, topics, and coverage reports. See `PLAN.md` for the broader roadmap.

## Architecture

- `app/`: Next.js App Router pages.
- `components/`: Client and server UI components.
- `lib/types.ts`: Public data model types.
- `lib/data.ts`: Loads the compact shared catalog book/entity indexes and exposes lookup maps/helpers; per-book provenance is loaded on demand.
- `lib/catalog.ts`: Book sorting and keyword search helpers.
- `lib/semantic-search.ts`: Shared semantic-search text construction, vector scoring, query normalization, and generic hybrid ranking helpers.
- `lib/browse-data.ts` and `lib/browse-types.ts`: Typed access to `data/public/browse.json` for precomputed browse/search rows.
- `lib/award-region.ts`: Region/country classification helpers for award programs.
- `lib/imprint-logos.ts`: Imprint logo asset resolution helpers.
- `lib/topics.ts`: Topic lookup and typing helpers.
- `app/api/search/semantic/route.ts`: Server-side OpenAI-backed semantic search endpoint used by the catalog UI.
- `components/use-semantic-book-search.ts`: Client hook for debounced semantic book search requests.
- `scripts/build-data.ts`: Builds compact shared catalog artifacts plus on-demand per-book details from source inputs and enrichment/curation patches.
- `scripts/build-semantic-index.ts`: Builds `data/public/book-semantic-index.json` from the public catalog using OpenAI embeddings.
- `scripts/build/`: Shared build helpers for award programs, curation, paths, text normalization, title resolution, and browse-data generation.
- `scripts/enrich-books.ts`: Book metadata completion from Open Library and Google Books, with persistent attempt tracking.
- `scripts/enrich-summaries.ts`: Text-first summary/description enrichment from Open Library APIs, Google Books, and optional local Open Library dumps.
- `scripts/discover-isbns.ts`: ISBN-first Open Library work/edition matching that writes high-confidence ISBN patches and review reports.
- `scripts/cache-book-covers.ts`: Caches usable remote catalog cover thumbnails into `public/book-covers/` and rewrites generated cover patches to local static URLs.
- `scripts/enrich-wikipedia-books.ts`: Adds conservative Wikipedia book-page evidence and publisher evidence for books.
- `scripts/book-enrichment-priority.ts`: Shared lane and priority scoring for book enrichment queues and runners.
- `scripts/enrich-subject-categories.ts`: Captures raw Google Books/Open Library subject labels as evidence for subject scoring.
- `scripts/classify-topics.ts`: Embedding/LLM-assisted topic classification that writes generated topic enrichment and review reports.
- `scripts/classify-reader-traits.ts`: Classifies reader-facing traits and reader level from catalog text, award context, publisher/imprint signals, and generated metadata.
- `scripts/normalize-imprints.ts`: Converts curated raw catalog publisher strings into explicit imprint and parent-publisher patches.
- `scripts/report-duplicate-books.ts` and `scripts/report-publisher-imprints.ts`: QA reports for likely duplicate books and publisher/imprint issues.
- `scripts/import-award-records/`: Source importers for normalized award-history records. `helpers.ts` contains shared importer utilities (text normalization, registry reads, write helpers); `wikitable.ts` is a MediaWiki wikitext parser used by several importers.
- `sources/`: Source manifest, curation patches, enrichment patches, taxonomy definitions, imprint normalization mappings, and the starter workbook.
- `sources/enrichment/`: Generated or curated metadata patches for books, awards, publishers, imprints, and sources.
- `data/raw/award-records/`: Source-backed raw award appearances before public catalog build.
- `data/public/`: Generated app-consumed data artifacts: compact catalog indexes, `book-details/`, `browse.json`, and the semantic manifest/vector pair.
- `data/reports/`: Generated QA reports, review queues, taxonomy reports, and analysis summaries (e.g. `topic-summary.json`, `import-report.json`).
- `data/cache/`: Gitignored pipeline provider caches and attempt ledgers (see `data/cache/README.md`). Safe to regenerate; attempt ledgers should be deleted only when deliberately resetting enrichment attempts.
- `public/award-logos/`, `public/imprint-logos/`, and `public/icons/`: Local UI assets for award marks, imprint marks, and retailer icons.
- `public/book-covers/`: Local cached cover thumbnails generated from acceptable remote catalog cover URLs.

## Current Snapshot


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [benjaminbreen/BookPrizeIndex](https://github.com/benjaminbreen/BookPrizeIndex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
