---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Static marketing/info website for the Frauenverein Sarmenstorf (a Swiss women's association),
built with **Astro** and deployed as a static site to `new.frauenverein-sarmenstorf.ch` (see
`CNAME`).

## Dev commands

```bash
npm run dev       # astro dev
npm run build     # astro build → dist/
npm run preview   # preview the production build
```

No test suite, linter config, or TypeScript build step beyond Astro's own `astro check`
(not configured as a script). `tsconfig.json` extends `astro/tsconfigs/strict` and references
`worker-configuration.d.ts` / `generate-types` (a `wrangler types` script exists in
`package.json`), but there is no `wrangler.toml` in the repo — these are leftovers and not
part of the current deployment flow.

Requires a `.env` (see `.env.example`) with `PUBLIC_SUPABASE_URL` / `PUBLIC_SUPABASE_ANON_KEY`
for the Supabase project — without it, `EventList`, the Galerie page, and `/admin` fail at
runtime (build still succeeds since data loading is client-side).

## Architecture

- **`src/layouts/BaseLayout.astro`** — the single shared page shell. Every page wraps its
  content in `<BaseLayout title="..." description="..." ogImage="...">`, which renders
  `<head>` (meta/OG tags, canonical URL via `Astro.site` from `astro.config.mjs`), plus the
  shared `Header` and `Footer` components and imports global `src/styles/base.css`.
- **`src/components/header.astro`** — site nav is defined in a single `navItems` array
  (with optional `children` for dropdowns) and rendered twice from that array: once for the
  desktop nav (with `<ul class="sub">` dropdowns) and once for the mobile overlay menu. The
  mobile burger/overlay open-close logic is a vanilla inline `<script>` at the bottom of this
  file. **When adding/removing a page that should appear in navigation, edit `navItems` once**
  — both menus update from the same source.
- **Per-page structure**: each `src/pages/*.astro` file imports `BaseLayout`, an optional
  page-specific stylesheet from `src/styles/<page>.css`, and wraps a `<section class="section">`
  / `<div class="container">` content block. Styling is per-page CSS files, no shared component
  library beyond `header`/`footer`/`EventList`.
- **`src/components/EventList.astro`** — events/agenda system shared across the homepage,
  `anlaesse`, and `jahresprogramm` pages. Data lives in Supabase (`public.events`, see
  "Supabase backend" below), not in a local JSON file.
  - The component renders an empty container server-side, then a client `<script>` fetches all
    rows from `events` (ordered by `date_iso`), builds `.event-card` elements (each with
    `data-date`), filters to only `date_iso >= today`, optionally truncates to the `limit`
    prop, and shows an "Aktuell sind keine bevorstehenden Anlässe erfasst." message if nothing
    remains.
  - `text_html` is inserted via `innerHTML` (trusted content, authored by admins in `/admin`),
    so it can contain raw HTML (links, `<strong>`, `<span style="...">` for color/uppercase).
  - Any keys in the per-event `extra` jsonb column are rendered generically as `"Key: value"`
    lines — this is how admins add ad-hoc fields (e.g. a start time) without schema changes.
  - `variant` prop (`'preview' | 'grid' | 'list'`) only changes the wrapper CSS class
    (`events-list` vs `events-grid events-grid--<variant>`) — filtering logic is identical.
- **`src/pages/galerie.astro`** — fetches `public.gallery_albums` and `gallery_images`
  client-side, resolves each `storage_path` to a public URL via
  `supabase.storage.from(GALLERY_BUCKET).getPublicUrl()`. For each album ("Anlass") that has
  images, renders a `.gallery-album` section (title + optional description) containing a
  `.carousel`/`.carousel-track` — the track is the image list duplicated once and animated via
  the `gallery-scroll` CSS keyframe (`translateX(0)` → `translateX(-50%)`, linear, infinite),
  which gives a seamless right-to-left looping carousel; duration scales with image count
  (`images.length * 6s`). Images with `album_id IS NULL` render below the albums in the original
  `.gallery` grid. Shared lightbox (click to enlarge, `Esc`/backdrop click to close) works for
  both carousel and grid images.
- **Forms** (e.g. `freizeitplausch_form.astro`) submit to **Formspree** (`action="https://formspree.io/f/..."`)
  via a `fetch` POST with `FormData` in an inline `<script>`, with client-side validation,
  a hidden `message` field built by a `buildSummary()` function that concatenates form values
  into a readable text block, and a success `.fp-modal` shown on success instead of a redirect.
- **`public/`** — static assets referenced by absolute path (`/images/...`, `/dokumente/...pdf`,
  `/js/...`). PDFs in `public/dokumente/` are linked directly from event `text_html` entries.

## Supabase backend

This site is otherwise static, but content for Jahresprogramm and Galerie lives in a Supabase
project. All app tables/functions live in the default **`public`** Postgres schema —
`src/lib/supabase.ts` creates a plain client with no `db.schema` override.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sky-Walker-xlsr/frauenverein-sarmenstorf](https://github.com/Sky-Walker-xlsr/frauenverein-sarmenstorf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
