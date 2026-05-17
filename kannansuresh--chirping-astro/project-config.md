---
trigger: always_on
description: <!-- markdownlint-disable-file -->
---

<!-- markdownlint-disable-file -->

# Agents.md — Chirping Astro

> Agentic development guide for the `kannansuresh/chirping-astro` repository.
> This file documents project conventions, architecture, key files, and
> task-specific guidance for AI agents working on this codebase.

---

## Project Overview

**Chirping Astro** is a Chirpy-inspired, multilingual, statically generated blog
theme built on **Astro 6.x**. It targets technical writers who need a fast,
accessible blog with first-class i18n, dark mode, MDX authoring, and zero
server-side runtime.

Markdown authoring also includes two custom remark code-block transforms:

- `ashtml` → raw HTML rendering
- `alert` → daisyUI alert markup with icon, title/description, and variant classes

| Dimension | Value |
|-----------|-------|
| Framework | Astro 6.x (static output) |
| Package manager | **Bun ≥ 1.1.0** (only supported PM; lockfile is `bun.lock`) |
| Styling | Tailwind CSS v4 + daisyUI v5 |
| Languages | TypeScript (strict), Astro, MDX, CSS |
| Locales | User-defined. Ships with `en` + `fr`; any locale can be added or set as default |
| Default locale | Always served at the URL root (no prefix). Configurable in `src/config.ts` |
| Search | Pagefind (static index, built post `astro build`) |
| Comments | Giscus (optional, GitHub Discussions backed) |
| Math | KaTeX (opt-in per post via `math: true` frontmatter) |
| CI | GitHub Actions — `deploy.yml` (Pages) + `pr-checks.yml` |
| Syncing | Starter template is synced using `.github/workflows/sync-starter.yml`, driven by `.starter-include` |

---

## Repository Layout

```
.
├── .starter-include           # Defines which files/folders are synced to the starter template
├── astro.config.mjs           # Astro + integrations config
├── bunfig.toml                # Bun runtime config
├── eslint.config.js           # Flat ESLint config (zero warnings policy)
├── tsconfig.json
├── package.json
├── .env.example               # All recognised env vars with comments
├── public/
│   └── images/                # Static assets (avatar, OG default, etc.)
└── src/
    ├── config.ts              # ★ Primary knobs: SITE, NAV, SOCIALS, GISCUS
    ├── content.config.ts      # Zod schema — post + page frontmatter
    ├── env.d.ts               # Astro env type shim
    ├── components/
    │   └── islands/           # Client-hydrated Astro islands
    │       ├── ThemeToggle.astro
    │       ├── LanguageSwitcher.astro
    │       ├── SearchButton.astro
    │       ├── TableOfContents.astro
    │       ├── BackToTop.astro
    │       └── Giscus.astro
    ├── content/
    │   ├── pages/<locale>/    # About + Privacy pages — one folder per locale
    │   └── posts/<locale>/    # Blog posts (.md / .mdx) — one folder per locale
    ├── i18n/
    │   ├── ui.ts              # Per-locale UI string dictionaries
    │   ├── utils.ts           # localePrefix(), formatDate(), etc.
    │   └── index.ts           # Re-exports
    ├── plugins/
    │   ├── remark-ashtml.ts   # `ashtml` fenced code block -> raw HTML
    │   └── remark-alert.ts    # `alert` fenced code block -> daisyUI alert HTML
    ├── layouts/
    │   ├── BaseLayout.astro   # Shell: topbar, sidebar, footer, no-FOUC script
    │   ├── PageLayout.astro   # Wrapper for static pages
    │   └── PostLayout.astro   # Wrapper for blog posts (TOC, Giscus, etc.)
    ├── pages/                 # Default locale routes (no URL prefix — always)
    │   ├── index.astro
    │   ├── 404.astro
    │   ├── about.astro
    │   ├── archives.astro
    │   ├── search.astro
    │   ├── rss.xml.ts
    │   ├── page/[page].astro
    │   ├── posts/[...slug].astro
    │   ├── tags/{index,[tag]}.astro
    │   ├── categories/{index,[category]}.astro
    │   └── <locale>/          # One sub-folder per non-default locale (mirrors root)
    ├── styles/global.css      # Tailwind v4 entry + daisyUI themes + CSS tokens
    └── utils/
        ├── posts.ts           # Collection helpers: sort, filter, paginate
        ├── reading-time.ts
        └── seo.ts
```

---

## Environment Variables

All variables live in `.env` (local) or the hosting provider's build environment.
The file `.env.example` is the canonical reference — never commit `.env`.

| Variable | Required | Purpose |
|----------|----------|---------|
| `SITE_URL` | ✅ | Canonical origin (no trailing slash). Used for OG, RSS, sitemap, hreflang. |
| `BASE_PATH` | GitHub Pages only | Sub-path prefix (e.g. `/chirping-astro`). Leave empty for root hosting. |
| `PUBLIC_GITHUB_HANDLE` | Optional | Sidebar GitHub icon + `SITE.author.url` |
| `PUBLIC_GITHUB_REPO` | Optional | Repo slug for custom integrations |
| `PUBLIC_TWITTER_HANDLE` | Optional | Sidebar Twitter icon |
| `PUBLIC_CONTACT_EMAIL` | Optional | Sidebar email icon |
| `PUBLIC_GISCUS_ENABLED` | Optional | Master switch (`true`/`false`). Default: `false`. |
| `PUBLIC_GISCUS_REPO` | Giscus | `<handle>/<repo>` |
| `PUBLIC_GISCUS_REPO_ID` | Giscus | From giscus.app |
| `PUBLIC_GISCUS_CATEGORY` | Giscus | Discussion category name |
| `PUBLIC_GISCUS_CATEGORY_ID` | Giscus | From giscus.app |
| `CI_SKIP_AUTO_OG_IMAGE` | CI only | Skip Satori OG generation in PR builds |
| `CI_SKIP_RSS_SITEMAP` | CI only | Skip RSS + sitemap in PR builds |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kannansuresh/chirping-astro](https://github.com/kannansuresh/chirping-astro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
