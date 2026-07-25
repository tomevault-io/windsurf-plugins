---
trigger: always_on
description: PortalJS is a Next.js framework for building data portals and catalogs. This file teaches AI assistants the conventions, patterns, and idioms used across this repo.
---

# PortalJS — AI Development Guide

PortalJS is a Next.js framework for building data portals and catalogs. This file teaches AI assistants the conventions, patterns, and idioms used across this repo.

## Core concepts — three surfaces

Every data portal is built from three surfaces. The template and routes map directly onto them; reason in these terms:

1. **Home** (`/`, `pages/index.tsx`) — what the portal is + a search box that routes to the catalog.
2. **Catalog** (`/search`, `pages/search.tsx`) — find/browse datasets. Static = full-text filter over `datasets.json`; scales to a search backend (CKAN/Solr) via `/portaljs-connect-ckan`.
3. **Showcase** (`/@<namespace>/<slug>`, `pages/[owner]/[slug].tsx`) — one dataset: metadata, a `Table` preview, download/API, and a Views slot for charts/maps.

Datasets are `@`-namespaced so they never collide with content pages. See `docs/core-concepts`.

## Repo structure

```
packages/
  core/                — layout/UI components (@portaljs/core)
  ckan/                — CKAN catalog UI + React components (@portaljs/ckan)
  ckan-api-client-js/  — pure CKAN API client (@portaljs/ckan-api-client-js)
examples/              — reference implementations (read these before building)
.claude/
  commands/     — Claude Code slash commands (OSS skills)
  datopian/     — Datopian-internal skills (require API keys)
  AUTHORING.md  — how to write new skills
```

## Component selection

The template ships its own lightweight components in `components/`. Do not add `@portaljs/components` — it bundles leaflet, vega, ag-grid, and pdf.js into a single non-tree-shakeable bundle and is not kept up to date with the local source.

| Need | Where |
|------|-------|
| Show tabular data (CSV/TSV/JSON) | `components/Table.tsx` — uses papaparse + @tanstack/react-table |
| Charts | Add a chart library directly (e.g. recharts, victory) — do not use @portaljs/components |
| Map (GeoJSON) | Add react-leaflet + leaflet directly if needed |
| Page layout, nav | Plain Tailwind — no layout package needed |
| CKAN catalog | `@portaljs/ckan` only if connecting to a CKAN backend |

## Data loading

**CSV or TSV from a local file:**
1. Place the file in `/public/data/filename.csv`
2. In the page: `<Table url="/data/filename.csv" />`

No server-side code needed. Next.js serves `/public/` statically. The `Table` component fetches via `url` prop using the browser's `fetch`.

**CSV string (inline data):**
```tsx
<Table csv={csvString} />
```

**JSON array:**
Pass as `data` prop:
```tsx
<Table data={rows} cols={[{ key: 'name', name: 'Name' }, ...]} />
```

**Remote URL (CORS-enabled):**
```tsx
<Table url="https://example.com/data.csv" />
```

**CKAN datastore:**
Use `datastoreConfig` prop — requires a running CKAN backend.

**Large files (>5MB):** Add a note in the page that loading may be slow. Consider server-side pagination via `datastoreConfig`.

## Page structure

Datasets themselves are manifest-driven and render via the showcase route (see Routing). A standalone content page looks like:

```tsx
// pages/about-the-data.tsx — datasets render via pages/[owner]/[slug].tsx from datasets.json
import { Table } from '../../components/Table'
import Head from 'next/head'

export default function DatasetPage() {
  return (
    <>
      <Head><title>Dataset Name</title></Head>
      <main className="max-w-5xl mx-auto px-4 py-8">
        <h1 className="text-3xl font-bold mb-6">Dataset Name</h1>
        <Table url="/data/filename.csv" />
      </main>
    </>
  )
}
```

**`_app.tsx`:**
```tsx
import '../styles/globals.css'
import type { AppProps } from 'next/app'

export default function App({ Component, pageProps }: AppProps) {
  return <Component {...pageProps} />
}
```

**`styles/globals.css` — Tailwind directives:**
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

## Routing conventions

The three surfaces (see Core concepts):

- `/` — **home** (`pages/index.tsx`): hero + search box → `/search`
- `/search` — **catalog** (`pages/search.tsx`): full-text list over `datasets.json`
- `/@<namespace>/<slug>` — **showcase** (`pages/[owner]/[slug].tsx`): one dataset, manifest-driven
- `/api/` — server-side API routes (avoid for simple portals; use static data in `/public/`)

Datasets are `@`-namespaced so they never collide with content pages. Pick one mode per portal — `theme` (single publisher, group by subject) or `owner` (multi-publisher, group by who published) — set via `NAMESPACE_TYPE` in `lib/datasets.ts`. Slug: lowercase, hyphenated, from the filename: `country-codes.csv` → `/@reference/country-codes`.

## Styling

- Tailwind CSS everywhere. Always include `@tailwindcss/typography` for prose content.
- Do not use inline styles. Do not use CSS modules unless the project already uses them.

Standard `tailwind.config.js`:
```js
module.exports = {
  content: [
    './pages/**/*.{js,ts,jsx,tsx}',
    './components/**/*.{js,ts,jsx,tsx}',
  ],
  theme: { extend: {} },
  plugins: [require('@tailwindcss/typography')],
}
```

## Standard `package.json` dependencies

Minimal portal:
```json
{
  "dependencies": {
    "next": "^14.0.0",
    "react": "^18.0.0",
    "react-dom": "^18.0.0",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [datopian/portaljs](https://github.com/datopian/portaljs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
