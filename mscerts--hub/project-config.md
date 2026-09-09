---
trigger: always_on
description: > For AI coding agents working on this repository.
---

# Agent Instructions — Microsoft Certification Hub

> For AI coding agents working on this repository.
> Keep this file updated as the project evolves.

---

## Identity

You are working on **msfthub.com** — a community site providing free study-material collections for Microsoft certification exams. Be direct, efficient, and preserve the existing code style.

If you are an authorized agent, you will have access to the tasks repository which you should refer to.

---

## Project Context

- **Repo:** https://github.com/mscerts/hub
- **Site:** https://msfthub.com
- **Stack:** Astro 6 + Starlight (`@astrojs/starlight` ^0.39), MDX, Tailwind
- **Package manager:** pnpm (do not use npm or yarn)
- **Build:** `pnpm build` = `astro check && astro build --force` → static output in `dist/` (`--force` clears the content-layer cache; see Build & Verify)
- **Preview:** `pnpm dev` for local development

---

## Directory Structure

```
src/
├── components/
│   ├── sections/              # Landing page sections (features, testimonials, navbar&footer, misc)
│   ├── ui/
│   │   ├── starlight/        # Starlight component overrides (Head, SiteTitle, TableOfContents, etc.)
│   │   ├── banners/          # AnnouncementBanner, CookieConsentBanner
│   │   └── modals/           # CookieConsentModal
│   ├── WikiList.astro        # Auto-generates wiki cards by directory prefix
│   ├── LabList.astro         # Auto-generates lab index cards from labs/<area>/ pages
│   ├── VoucherList.astro     # Auto-generates voucher cards by voucherCategory frontmatter
│   ├── MarkdownContent.astro # Custom markdown wrapper
│   └── PartnerBadge.astro
├── content/
│   ├── docs/
│   │   ├── aibusiness/       # AI Business exam pages (AB-*)
│   │   ├── azure/            # Azure exam pages (AZ-*, AI-*, DP-*)
│   │   ├── dynamics/         # Dynamics 365 exam pages (MB-*)
│   │   ├── github/           # GitHub exam pages (GH-*)
│   │   ├── microsoft365/     # M365 exam pages (MS-*, MD-*)
│   │   ├── power/            # Power Platform exam pages (PL-*)
│   │   ├── security/         # Security exam pages (SC-*)
│   │   ├── guide/            # Certification program guide (8 pages)
│   │   ├── prepare/          # How-to-prepare pages (5 pages)
│   │   ├── vouchers/         # Discounted exam voucher pages
│   │   ├── labs/             # Lab collections (per-exam lab pages, parallel to exam structure)
│   │   ├── labs.mdx          # Lab index page (uses LabList component)
│   │   └── wiki.mdx          # Wiki index page (uses WikiList component)
│   └── blog/                 # Blog collection (authors, pubDate, cardImage, readTime, tags)
├── data_files/
│   ├── constants.ts          # Site metadata (title, description, SEO, OG)
│   ├── faqs.json
│   ├── features.json
│   └── mega_link.ts
├── content.config.ts         # Content schemas (docs + blog collections)
astro.config.mjs              # Sidebar, redirects, integrations, component overrides
```

## Content Collections

### `docs` collection
Extends Starlight's `docsSchema` with one custom field:
```ts
voucherCategory: z.enum(["100%", "50%", "Special"]).optional()
```
Used by `VoucherList.astro` to auto-categorize voucher pages.

### `blog` collection
Schema: `title`, `description`, `authors` (array with name/image), `pubDate`, `updatedDate?`, `cardImage`, `cardImageAlt?`, `readTime`, `tags?`, `draft?`

---

## Conventions

### Exam Pages
- One MDX file per exam at `src/content/docs/<area>/<CODE>.mdx`
- Routes are lowercase: `/azure/az-800/`, `/power/pl-300/`
- Filenames are uppercase: `AZ-800.mdx`, `PL-300.mdx`
- `<area>` ∈ `aibusiness | azure | dynamics | github | microsoft365 | power | security`
- Area mapping: `azure` = AZ-*, AI-*, DP-*; `aibusiness` = AB-*; `dynamics` = MB-*; `github` = GH-*; `microsoft365` = MS-* and MD-*; `power` = PL-*; `security` = SC-*.
- Astro collection IDs and rendered routes are lowercase regardless of filename case (`AZ-800.mdx` → id `azure/az-800`, route `/azure/az-800/`), so changing only a filename's case never changes its URL and needs no redirect. It does, however, poison Astro's incremental content-layer cache: the entry keeps the same ID and content digest, so the loader reuses the cached *old* `filePath`, which then fails to resolve on case-sensitive Linux (Cloudflare restores `node_modules/.astro` between builds). `pnpm build` passes `--force` to clear that cache, which is what makes such renames safe.
- Frontmatter title is `<CODE> Study Materials`.
- Frontmatter description must use the exact template: `Collection of study materials for the certification exam <CODE>: <Exam Name>. Contains official Microsoft Learn materials, labs, videos, practice tests and paid courses.`
- Use the official exam name, not the certification name. Verify it against Microsoft Learn; these names often differ.
- Exam pages do not use `voucherCategory`.

### URL Formats (normalized — preserve these)
- **Exam (preferred when published):** `https://learn.microsoft.com/credentials/certifications/exams/<code>?WT.mc_id=studentamb_165290`
- Some beta/new exams use a certification slug instead of `/exams/<code>`; use Microsoft's verified current destination rather than forcing a URL pattern.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mscerts/hub](https://github.com/mscerts/hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
