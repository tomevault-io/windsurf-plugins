---
trigger: always_on
description: This is **plain**, a Git-native CMS: content is Markdown files, configuration is JSON, the build emits a static site into `dist/`. The full spec is `cms-spec.md` — read it before structural changes. This file is the working reference for day-to-day edits.
---

# CLAUDE.md — how to work on this repo

This is **plain**, a Git-native CMS: content is Markdown files, configuration is JSON, the build emits a static site into `dist/`. The full spec is `cms-spec.md` — read it before structural changes. This file is the working reference for day-to-day edits.

## The one rule

**Before committing, always run both:**

```sh
node --test tests/
node build.js
```

A red test or a failed build must never be committed. The golden-file test compares built output byte-for-byte; if you intentionally changed output, run `node tests/update-goldens.js` and review the diff.

## Hard constraints (from cms-spec.md §2 — never violate)

- Vanilla only: no frameworks, no bundlers, no TypeScript. Plain ES modules, JSDoc for types.
- Exactly one runtime dependency: `marked`. No new packages, including dev dependencies.
- Core (`build.js` + `lib/` + admin JS) stays under 2,500 lines; no file over 400. Too big → make it a plugin.
- No database. All state is files in this repo.
- The published site must work with JavaScript disabled.
- `lib/util.js`, `lib/template.js`, `lib/markdown.js` are **isomorphic**: they must never import `node:*` — the admin runs them in the browser so previews match the build exactly.

## Commands

| Command | What it does |
| ------- | ------------ |
| `node build.js` | Build the site into `dist/` |
| `node build.js --watch` | Build, serve on :4000, rebuild on change |
| `node --test tests/` | Run the test suite |
| `node tests/update-goldens.js` | Regenerate golden files after an intentional output change |

## Content model

Collections are defined in `site.config.json`. A collection = a folder of `.md` files + a field schema:

```json
"posts": {
  "path": "content/posts",        // folder of .md files
  "urlPattern": "/blog/:slug/",   // must start with / and contain :slug
  "template": "post",             // theme template for one item
  "listUrl": "/blog/",            // optional: emit a paginated list page
  "listTemplate": "list",         // required if listUrl is set
  "label": "Blog",                // optional: heading for list pages
  "sortBy": "date", "sortOrder": "desc",
  "pageSize": 10,                 // pagination size for list pages
  "rss": true,                    // emit <listUrl>rss.xml
  "fields": [
    { "name": "title", "type": "text", "required": true },
    { "name": "date",  "type": "date", "required": true },
    { "name": "draft", "type": "boolean", "default": false }
  ]
}
```

Field types: `text`, `textarea`, `date`, `boolean`, `image`, `list`, `select` (needs `options`). The admin renders its edit forms from this schema, so **adding a field to config is the whole job** — no code changes.

**`render: false`** (optional) makes a **data-only collection**: scanned, schema-validated, sorted, and admin-editable exactly like any other, and exposed to templates as `collections.<name>`, but it emits **no** item pages or list page — so it needs no `urlPattern`, `template`, or `listUrl`. Use it for repeated home-page sections (feature cards, FAQ entries, testimonials, pricing tiers) where each entry is one editable `.md` file with no URL of its own. Items still get `api/<name>/…` JSON (their `url` is `null`) but are excluded from the sitemap, search index, and `llms.txt`. Order them with a numeric `order` field plus `"sortBy": "order", "sortOrder": "asc"`; render them with `{{#each collections.<name> as item}}…{{/each}}` in the page template that needs them.

**`site.basePath`** (optional): serve under a subpath, e.g. GitHub *project* Pages at `/<repo>/`. The build prefixes every root-relative `href`/`src` (and redirect target) with it; set `site.url` to the full base too (`https://user.github.io/<repo>`). Leave it empty (default) for a site served at the domain root — user Pages, Cloudflare/Netlify, or a custom domain.

**`site.oauthUrl`** (optional): the deployed OAuth Worker URL (`workers/oauth/`). When set, the admin sign-in screen shows a **"Sign in with GitHub"** button (the paste-a-token form moves under "or use an access token"); writers with repo write access click it, authorize once, and publish — no PAT. Leave it out for token-only sign-in (v1). The admin opens `<oauthUrl>/login` in a popup and accepts the token only from a `postMessage` matching that origin.

**`site.favicon` / `site.appleTouchIcon` / `site.socialImage`** (optional): per-site branding without editing a theme. `favicon` swaps the icon link away from the theme's `/assets/favicon.svg`; `appleTouchIcon` adds an iOS home-screen icon link; `socialImage` is the site-wide share image — every theme emits it as `og:image` + `twitter:card` for pages that have no `cover`. Values are root-relative paths served from `media/` (e.g. `/media/brand/og-image.png`); meta tags absolutize them with `site.url`. The project's brand kit (SVG masters + generator + platform rasters) lives in the site repo, `victorantos/plain-cms.com`, under `assets/` — this engine repo ships only the default favicon and the README logos in `.github/`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [plain-cms/plain](https://github.com/plain-cms/plain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->
