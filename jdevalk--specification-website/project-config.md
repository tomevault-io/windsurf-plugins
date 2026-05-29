---
trigger: always_on
description: This file is read by Claude Code (and any other agent that respects the convention) before it works on this repo. Keep it short, accurate, and updated alongside the code it describes.
---

# CLAUDE.md — repo guidance for AI agents (and humans skimming)

This file is read by Claude Code (and any other agent that respects the convention) before it works on this repo. Keep it short, accurate, and updated alongside the code it describes.

## What this project is

`specification.website` — a platform-agnostic specification of what a good website does. Static Astro site, deployed to Cloudflare Pages from this repository's `main` branch. MIT licensed (code) / CC BY 4.0 (content).

Live: <https://specification.website> and <https://specification-website.pages.dev>.

## Top-level rule: everything is derived from the content collection

The whole site is generated from one source of truth: Markdown files under `src/content/spec/<category>/<slug>.md`, with the schema in `src/content.config.ts` and the category list in `src/lib/site.ts`.

**When you add, remove, or change a spec page, the following surfaces update automatically on the next build. Do not hand-edit any of them:**

- `/checklist/` — every spec entry, grouped by category, tickable. Built by `src/pages/checklist.astro` from `getCollection('spec')`.
- `/spec/` (index) and `/spec/<category>/` (category indexes). Same source.
- `/spec/<category>/<slug>/` — the HTML page. Built by `src/pages/spec/[category]/[slug].astro`.
- `/spec/<category>/<slug>.md` — the raw Markdown endpoint with YAML frontmatter. Built by `src/pages/spec/[category]/[slug].md.ts`.
- `/llms.txt` and `/llms-full.txt` — the agent-facing indexes. Built by `src/pages/llms.txt.ts` and `src/pages/llms-full.txt.ts`.
- `/sitemap-index.xml` — built by `@astrojs/sitemap`.
- `/rss.xml` — built by `src/pages/rss.xml.ts`.
- The Pagefind search index — built by `pagefind --site dist` in the `build` script. Powers the `/search/` page and the global ⌘K overlay.

**If you find yourself editing the checklist by hand, you are doing it wrong.** Edit the spec entry's front matter (`status`, `summary`, `title`) and rebuild.

## Status discipline

The status field is the user-facing contract. Use it precisely.

- **`required`** — the web platform contract breaks, or a clear class of users is harmed, without it. Examples: `<title>`, `<meta charset>`, HTTPS, image alt text, custom 404 returning 404.
- **`recommended`** — a modern site should do it. Examples: CSP, HSTS, structured data, Open Graph, security.txt, /llms.txt.
- **`optional`** — depends on context. Examples: image sitemaps, OpenID Configuration, IDN support.
- **`avoid`** — outdated, harmful, or superseded by a working alternative. Examples: soft-404, empty links/buttons.

When in doubt, default to `recommended`, not `required`. The bar for `required` is "the platform breaks without it", not "we strongly suggest it".

## Cardinal rules for content

These mirror `CONTRIBUTING.md`. Enforce them in your own writing and when reviewing.

1. **Cite primary sources.** Every spec page needs 2–4 sources in front matter, weighted toward standards bodies: WHATWG, W3C, IETF RFCs, IANA, WCAG, schema.org, sitemaps.org, llmstxt.org. Then MDN / web.dev / Google Search Central for practical context. Avoid blog posts and vendor marketing.
2. **Stay platform-agnostic.** Describe outcomes, not implementations. "Set `Content-Security-Policy`" is in scope. "Add this to your `next.config.mjs`" is not. Link out to platform docs instead.
3. **Be honest about status.** If something is shipping as `required` but the platform works without it, downgrade to `recommended`. If the source URL is dead, replace it (Wayback Machine is acceptable) or remove the citation.
4. **British English.** "colour", "behaviour", "internationalisation", "licence" (noun).
5. **Section structure.** `## What it is`, `## Why it matters`, `## How to implement`, `## Common mistakes`, `## Verification`. Last two are optional if they would not add value.
6. **Length.** 250–500 words of body content. Be useful, not padded.

## Architecture quick reference

| Path | Purpose |
|---|---|
| `src/content/spec/<cat>/<slug>.md` | Spec content. Edit here. |
| `src/content.config.ts` | Content collection schema. Edit if adding a field. |
| `src/lib/site.ts` | Site metadata + the canonical category list. |
| `src/layouts/BaseLayout.astro` | HTML shell, head, dialog for ⌘K search, Plausible (PROD only). |
| `src/layouts/SpecLayout.astro` | Spec page wrapper; emits TechArticle + BreadcrumbList JSON-LD; advertises Markdown alt via `markdownUrl`. |
| `src/components/HeadMeta.astro` | `<head>` metadata, canonical, OG, Twitter, JSON-LD, RSS / sitemap / markdown alternates. |
| `src/components/SiteHeader.astro` | Header nav. Contains the ⌘K trigger. |
| `src/components/SiteFooter.astro` | Footer. Privacy / search links live here. |
| `src/pages/spec/[category]/[slug].astro` | The dynamic HTML route. |
| `src/pages/spec/[category]/[slug].md.ts` | The dynamic Markdown route. |
| `src/pages/llms.txt.ts`, `src/pages/llms-full.txt.ts`, `src/pages/rss.xml.ts` | Derived endpoints. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jdevalk/specification.website](https://github.com/jdevalk/specification.website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
