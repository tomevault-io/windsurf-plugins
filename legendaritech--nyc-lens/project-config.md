---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

BBL Club is a Next.js 16 application for exploring NYC real estate data (ACRIS, PLUTO, DOB, HPD datasets). Built with React 19, TypeScript, Tailwind CSS v4, and ag-Grid Enterprise for complex data tables with server-side filtering/sorting via Elasticsearch.

## Development Commands

### Core Development
- `npm run dev` - Start development server with Turbopack (http://localhost:3000)
- `npm run build` - Production build (uses webpack; Turbopack is dev-only in Next.js 16)
- `npm start` - Start production server
- `npm run lint` - Run ESLint

### Testing
- `npm test` - Run all tests (unit + Storybook) with Vitest
- `npm run test:watch` - Run tests in watch mode
- Single test file: `npx vitest run path/to/test.test.tsx`
- Test a specific component: `npx vitest run -t "component name"`

### Storybook
- `npm run storybook` - Start Storybook dev server on port 6006
- `npm run build-storybook` - Build static Storybook

## Architecture Overview

### Application Structure

```
src/
├── app/                        # Next.js App Router pages
│   ├── api/                    # API routes (server-side only)
│   │   └── acris/              # ACRIS data endpoints (properties, documents, parties)
│   ├── property/[bbl]/         # Property detail pages (dynamic BBL routes)
│   │   ├── dob/                # DOB data tabs (violations, permits, jobs, etc.)
│   │   ├── hpd/                # HPD data tabs (violations, registrations, permits)
│   │   ├── pluto/              # PLUTO property data
│   │   └── tax/                # Tax assessment data
│   ├── dob/                    # DOB dataset exploration pages
│   ├── hpd/                    # HPD dataset exploration pages
│   └── search/                 # Property search interface
├── components/
│   ├── ui/                     # Reusable primitives (Button, Tabs, Switch, Autocomplete)
│   ├── table/                  # ag-Grid table components (property, document, party)
│   ├── search/                 # Search-specific components (PropertyAutocomplete, HighlightedText)
│   ├── layout/                 # Layout components (ResizableSidebarLayout, SidebarNav)
│   └── icons/                  # Icon components
├── data/                       # Data access layer (server-only)
│   ├── elasticsearch.ts        # Elasticsearch client
│   ├── db.ts                   # MSSQL database connection
│   ├── pluto.ts                # PLUTO data queries
│   ├── valuation.ts            # Tax valuation queries
│   ├── dobJobs.ts              # DOB jobs data
│   └── dobViolations.ts        # DOB violations data
├── types/                      # TypeScript type definitions
│   ├── acris.ts                # ACRIS record types
│   ├── pluto.ts                # PLUTO field types
│   ├── dob.ts                  # DOB data types
│   ├── valuation.ts            # Tax assessment types
│   └── api.ts                  # API request/response types
├── utils/                      # Utility functions
│   ├── cn.ts                   # Tailwind class name merger (clsx + tailwind-merge)
│   ├── agGrid.ts               # ag-Grid ↔ Elasticsearch query builder
│   └── formatters.ts           # Data formatting utilities
└── constants/                  # Constants and lookup tables
    ├── acris.ts                # ACRIS document types, control codes
    ├── nyc.ts                  # Borough codes, BBL utilities
    └── landUse.ts              # Land use classifications
```

### Key Architectural Patterns

#### Server-Side Data Access
- **All data queries are server-side only** - Files in `src/data/` use `'server-only'` to prevent client bundling
- **Two data sources**:
  - Elasticsearch: ACRIS property records (properties, documents, parties)
  - MSSQL: NYC Open Data (PLUTO, tax assessments, DOB, HPD)
- **API routes** in `src/app/api/acris/` handle ag-Grid server-side requests, converting ag-Grid filter/sort models to Elasticsearch DSL via `buildEsQueryFromAgGrid()` in `src/utils/agGrid.ts`

#### ag-Grid Server-Side Model
- **PropertyTable** (`src/components/table/property/PropertyTable.tsx`): Main ACRIS property table
  - Uses `serverSideDatasource` to fetch data via `/api/acris/properties`
  - Supports filtering, sorting, pagination (10K row limit)
  - Master-detail rows expand to show related documents (DocumentTable)
- **DocumentTable**: Nested table showing ACRIS documents for a property
  - Fetches data via `/api/acris/documents`
  - Also uses server-side model
- **ag-Grid Enterprise license**: Required (configured in `PropertyTable.tsx:14`)

#### Next.js App Router Patterns
- **Server Components by default**: Use `'use client'` only when needed (hooks, events, browser APIs)
- **Dynamic routes**: `app/property/[bbl]/` uses BBL (Borough-Block-Lot) as route param
- **Parallel routes**: Property pages use tabs for different data sources (PLUTO, DOB, HPD, etc.)
- **Data fetching**: Server Components call `src/data/` functions directly; Client Components use API routes

#### Property Detail Pages (`app/property/[bbl]/`)
- **BBL format**: 10-digit string (1 borough + 5 block + 4 lot), e.g., `"1000010001"`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LegendariTech) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
