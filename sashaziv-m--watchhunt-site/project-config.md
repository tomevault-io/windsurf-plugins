---
trigger: always_on
description: > **ChronoHunt** is a watch marketplace aggregator with a comprehensive watch database. This document provides essential context for AI assistants working on the project.
---

# ChronoHunt Project Guide

> **ChronoHunt** is a watch marketplace aggregator with a comprehensive watch database. This document provides essential context for AI assistants working on the project.

## Quick Start

```bash
# Start PostgreSQL (Docker)
docker start watches-db

# Start development server
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/watches" npm run dev

# Open http://localhost:3000
```

## Project Structure

| Directory | Purpose |
|-----------|---------|
| `src/app/api/` | Next.js API routes (Prisma ORM) |
| `src/components/` | React components (WatchGrid, FilterSidebar, etc) |
| `src/context/` | FilterContext for filter state |
| `scripts/scraper/` | **Scraper engines and CLI** |
| `scripts/scraper/config/` | Brand configurations (100+ brands) |
| `scripts/scraper/engines/` | ShopifyScraper, HtmlScraper, SpaScraper |
| `scripts/scraper/scrapers/` | Custom brand-specific scrapers |
| `scripts/scraper/utils/` | Parsers (movement, color, specs) |
| `prisma/` | Database schema (PostgreSQL) |
| `data/` | JSON files (accessories, movements, static-filters) |
| `tests/unit/` | Unit tests (Vitest) |
| `.claude/agents/` | Claude Code agent prompts |
| `.claude/commands/` | Claude Code slash commands |

## Tech Stack

- **Frontend:** Next.js 16 + React 19 + TailwindCSS 4
- **Backend:** Next.js API Routes + Prisma ORM
- **Database:** PostgreSQL (Docker)
- **Scraping:** Custom engines (Shopify JSON, HTML, SPA, Custom)
- **Testing:** Vitest (145+ tests passing)

---

## Scraper Architecture (v2)

The scraper system uses a **configuration-driven engine pattern**. Instead of 15+ unique scraper files, brands are defined by type and config.

### Brand Configuration

Location: `scripts/scraper/config/brand-configs.ts`

```typescript
// 100+ brands configured across 4 engine types
export const brandConfigs: Record<string, BrandConfig> = {
    'san-martin':  { type: 'CUSTOM', scraperPath: './scrapers/san-martin.js', ... },
    'squale':      { type: 'SHOPIFY', baseUrl: 'https://www.gnomonwatches.com', ... },
    'helm':        { type: 'SPA', collectionUrl: '/watches.html', waitForSelector: '.wsite-multicol-col', ... },
    'wise':        { type: 'SHOPIFY', baseUrl: 'https://wisetimepiece.com', ... },
};
```

### Scraper Engines

| Engine | Use Case | Dependencies |
|--------|----------|--------------|
| `ShopifyScraper` | Shopify stores (most brands) | fetch + Cheerio |
| `HtmlScraper` | Static HTML sites | fetch + Cheerio |
| `SpaScraper` | JS-heavy sites, WooCommerce, Weebly | Puppeteer |
| `CUSTOM` | Brand-specific logic | Per-scraper |

### SPA Scraper Features

- **Selector validation**: Warns if `productItem` selector matches >100 elements
- **Anti-bot measures**: Puppeteer stealth, random delays, Cloudflare handling
- **Lazy image loading**: Checks `data-src`, `srcset` attributes
- **Auto-scroll**: Loads lazy content before parsing

### Custom Scrapers

| Scraper | Target | Notes |
|---------|--------|-------|
| `islander.ts` | longislandwatch.com | BigCommerce, parses #tab-warranty |
| `squale-liw.ts` | longislandwatch.com/squale | Same pattern as Islander |
| `san-martin.ts` | sanmartin.watch | Custom variant handling |
| `nodus.ts` | noduswatches.com | NodeX clasp detection |

### CLI Usage

```bash
# Scrape all brands
npx tsx scripts/scraper/cli.ts --verbose

# Scrape specific brand
npx tsx scripts/scraper/cli.ts --brand cronos --verbose

# Dry run (no DB writes)
npx tsx scripts/scraper/cli.ts --brand san-martin --dry-run

# Clean and rescrape (deletes existing brand data first)
npx tsx scripts/scraper/cli.ts --brand squale --clean

# Scrape by engine type
npx tsx scripts/scraper/cli.ts --type SHOPIFY
```

### GitHub Actions

Scheduled scraping runs daily at 6 AM UTC via `.github/workflows/scraper.yml`. Requires `DATABASE_URL` secret in GitHub repository settings.

---

## Database

### Schema (PostgreSQL via Prisma)

| Model | Fields | Notes |
|-------|--------|-------|
| `Watch` | brand, model, reference, specs... | Main watch records |
| `Variant` | watchId, color, imageUrl, variantUrl | Normalized from JSON blob |

### Current Stats (2026-01-15)

| Metric | Value |
|--------|-------|
| **Total Watches** | 7,220 |
| **Configured Brands** | 101 |
| **Unit Tests** | 145+ passing |

**Top Brands:**
- Zelos (472), Sugess (312), Spinnaker (280), Kuoe (248)
- Seiko (243), Citizen (241), Orient (237), Addiesdive (210)
- Sidereus (180), Henry Archer (170), San Martin (167)

### Key Commands

```bash
# Generate Prisma client
npx prisma generate

# Push schema to DB
npx prisma db push

# Open Prisma Studio
DATABASE_URL="..." npx prisma studio

# Count watches
npx tsx -e "import{PrismaClient}from'@prisma/client';new PrismaClient().watch.count().then(console.log)"

# Regenerate static filters (REQUIRED after scraping)
npx tsx scripts/generate-static-filters.ts
```

---

## Testing

Unit tests are in `tests/unit/` and run with Vitest:

```bash
# Run all tests
npm test

# Run specific test file
npx vitest run tests/unit/database-integrity.test.ts
```

### Test Files

| File | Coverage |
|------|----------|
| `parser-functions.test.ts` | All 12 parser utility functions |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sashaziv-m/watchhunt-site](https://github.com/sashaziv-m/watchhunt-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
