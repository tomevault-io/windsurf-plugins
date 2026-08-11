---
trigger: always_on
description: Index — SSR SEO for public routes (meta tags, canonical URLs, JSON-LD, CMS copy, audits). See web-seo-*.mdc rules for detail.
---


# Web: SSR SEO (index)

Public marketing and catalog routes under `web/src/routes/(public)/` plus the landing route (`web/src/routes/+page.*`) must ship **SSR metadata** in the initial HTML. Compute SEO in `+page.server.ts`; forward it through universal `+page.ts`; render in `+layout.svelte` (`MetaTags`) and `JsonLdHead`.

**Docs** use a separate head component: `DocsSeoHead.svelte` (same JSON-LD serialization helpers).

Root layout (`+layout.server.ts` → `baseMetaTags`) merges with page data in `+layout.svelte` via `deepMerge(data.baseMetaTags, page.data.pageMetaTags)`.

## Rule map

| Topic | Rule file |
| --- | --- |
| Meta tags, `createMetaData`, OG/Twitter | **web-seo-meta-tags** |
| Canonical URLs, query params, UTM, robots | **web-seo-canonical** |
| JSON-LD (`schema-dts`, builders, `JsonLdHead`) | **web-seo-jsonld** |
| Landing / FAQ CMS copy on the server | **web-seo-cms-copy** |
| Universal `+page.ts` forwarding on client nav | **web-sveltekit-universal-page-load** |
| Charset, rendering, pagination, mixed content, audits (this file) | **web-seo-index** |

## Common edge cases

| Mistake | Impact | Code fix (this repo) | Rule |
| --- | --- | --- | --- |
| Ignoring parameter handling | Crawl waste on filter/pagination variants | Path-only `buildCanonicalUrl`; param table by type | **web-seo-canonical** |
| Tracking params (`utm_*`, `gclid`, …) | Duplicate URLs, authority dilution | Canonical strip + `UtmAttribution.svelte` bar cleanup | **web-seo-canonical** |
| Content only after client JS | Thin or empty index | `export const ssr = true`; server `pageMetaTags` + `schemaData` | This file → Rendering |
| Infinite scroll on indexable hubs | Products/posts never indexed | Server pagination + `Pagination.svelte` with crawlable links | This file → Pagination |
| Mixed HTTP/HTTPS assets or URLs | Broken images, blocked subresources | HTTPS canonical/OG in `createMetaData`; `https://` for external assets | This file → Mixed content |
| Charset mismatch | Garbled international copy | `app.html` charset first; `ensureUtf8CharsetResponse` | This file → Charset |

## Audit playbook (manual — GSC / DevTools)

Run after shipping SEO changes or when Coverage/duplicate reports spike.

| Audit | Where | Pass criteria |
| --- | --- | --- |
| **Parameter handling** | GSC → Settings → URL parameters | UTM/`gclid`/`fbclid` marked “doesn't change page content” (optional); canonical on `?page=` variants points to path-only hub URL |
| **Rendering** | GSC → URL Inspection → View crawled page / Live test | Raw HTML and rendered HTML both include `<title>`, `<link rel="canonical">`, and `<script type="application/ld+json">` with matching URLs |
| **Indexation** | GSC → Pages / Coverage | Public hubs and detail URLs indexed; auth/app routes excluded (`noindex`, `robots.txt` Disallow) |
| **Mixed content** | DevTools → Security (on HTTPS deploy) | No “active mixed content”; no `http://` subresources on indexable pages |
| **Charset** | View page source + Network → document `Content-Type` | `<meta charset="utf-8">` is first in `<head>`; response header includes `charset=utf-8`; non-ASCII copy renders correctly |

Code-side rendering or canonical gaps → fix server loads per **web-seo-meta-tags**, **web-seo-canonical**, **web-seo-jsonld**, **web-seo-cms-copy**. Do **not** block parameterized URLs in `robots.txt` for consolidation (see **web-seo-canonical**).

## Charset (UTF-8)

Crawlers and browsers must decode HTML as **UTF-8**. A `<meta charset="UTF-8">` that does not match the real byte encoding (or a missing HTTP charset) makes non-English copy unreadable and hurts international SEO.

| Layer | Mechanism |
| --- | --- |
| **HTML** | `web/src/app.html` — `<meta charset="utf-8" />` is the **first** child of `<head>` (within the first 1024 bytes). Do not move it below styles, scripts, or `%sveltekit.head%`. |
| **HTTP** | `hooks.server.ts` — `ensureUtf8CharsetResponse` adds `charset=utf-8` when SvelteKit SSR returns `content-type: text/html` without a charset. |
| **Text feeds** | `robots.txt`, `rss.xml`, `llms*.txt`, markdown doc routes, and sitemap XML declare `charset=utf-8` (or `encoding="UTF-8"` in XML prologs). |
| **Source** | Keep repo source and CMS/API payloads UTF-8; avoid re-encoding user copy to Latin-1. |

## JavaScript rendering mismatch

Google separates **crawl**, **render**, and **index**. If meaningful content or SEO signals exist only after client JS runs, the indexer may store an empty or thin version even when users see the full page.

**This stack (SvelteKit SSR):** public routes use `export const ssr = true`, compute `pageMetaTags` and `schemaData` in `+page.server.ts`, and render them in the first HTML response (`MetaTags` in `+layout.svelte`, `JsonLdHead` in `+page.svelte`). Do not rely on `onMount`, `if (browser)`, or client-only presenter fetches for indexable copy or structured data.

| Signal | Must appear in initial HTML |
| --- | --- |
| `<meta charset="utf-8">` | First child of `<head>` in `app.html` (see **Charset** above) |
| `<title>`, meta description, OG/Twitter | `pageMetaTags` from server load |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ratimon/openquok-monorepo](https://github.com/Ratimon/openquok-monorepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
