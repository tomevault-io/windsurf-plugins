---
trigger: always_on
description: This is an informational website about **Miriam González Pérez**: a patient with metastatic HR+/HER2- breast cancer with neuroendocrine differentiation (~80%) and FGFR1 ×13 / CCND1 ×20 / 11q13 cluster amplification. The goal is to explain the case to the scientific community, raise funds for an advanced molecular rebiopsy (GoFundMe), and publicly document the process of a patient using AI and an international expert network to navigate precision oncology.
---

# CLAUDE.md

## About this Project

This is an informational website about **Miriam González Pérez**: a patient with metastatic HR+/HER2- breast cancer with neuroendocrine differentiation (~80%) and FGFR1 ×13 / CCND1 ×20 / 11q13 cluster amplification. The goal is to explain the case to the scientific community, raise funds for an advanced molecular rebiopsy (GoFundMe), and publicly document the process of a patient using AI and an international expert network to navigate precision oncology.

The website has two audiences: scientists/physicians (science page, molecular profile, clinical trials) and general public/donors (story, GoFundMe, timeline).

## Stack & Technical Decisions

### Framework: Nuxt 4 + SSG
- **Nuxt 4** with static generation (`pnpm run generate`). No backend. No database. No custom API.
- Deploy on Vercel/Netlify/Cloudflare Pages connected to GitHub. Every push to `main` triggers automatic redeployment.
- We chose SSG over SSR because content changes infrequently (weekly updates) and we want free serverless hosting.
- Nuxt 4 uses an `app/` directory for all source files (pages, components, assets, layouts).

### SEO: @nuxtjs/seo + @nuxtjs/sitemap + nuxt-ai-ready
- `@nuxtjs/seo` is a meta-module bundling: nuxt-og-image, nuxt-robots, nuxt-schema-org, nuxt-link-checker, nuxt-seo-utils.
- `@nuxtjs/sitemap` generates `/sitemap.xml` with i18n hreflang support — integrates automatically with `@nuxtjs/i18n`.
- `nuxt-ai-ready` generates `/llms.txt` for AI/LLM agent discoverability.
- **Module order matters**: `@nuxtjs/seo` and `@nuxtjs/sitemap` must be listed BEFORE `@nuxt/content` in `nuxt.config.ts`.
- `robots.txt` is managed by `nuxt-robots` — do not put a `robots.txt` in `public/`. The source file is `public/_robots.txt`.
- All pages use `useSeoMeta()` for meta tags. Do not use `useHead()` for SEO-related tags.
- OG images use `nuxt-og-image` via `defineOgImage()`. The template is `app/components/OgImage/Default.takumi.vue`.
- `site.url` and `site.name` in `nuxt.config.ts` feed all SEO modules — Nuxt SEO appends `site.name` to page titles automatically, so page titles should not include " | Miriam González" or " — Miriam González" suffixes.

### i18n: Spanish + English from Day 1
- `@nuxtjs/i18n` with `prefix_except_default` strategy (Spanish without prefix, English at `/en/`).
- UI strings live in `i18n/locales/es.json` and `i18n/locales/en.json`.

### Tailwind CSS
- Custom warm palette: `ink` (warm grays), `ocean` (deep blue for scientific content).
- Typography: **Fraunces** (serif, display/headings), **Source Sans 3** (sans, body), **JetBrains Mono** (mono, scientific data).
- `@tailwindcss/typography` plugin for prose on the story page.
- Custom utility classes in `app/assets/css/main.css`: `.section-container`, `.card-base`, `.tag-gold`, `.tag-ocean`, `.heading-display`, `.link-underline`, `.animate-fade-up`, `.stagger-children`.

### Icons
- `@nuxt/icon` with Phosphor Icons (prefix `ph:`). Examples: `ph:heart-fill`, `ph:flask-fill`, `ph:stethoscope-fill`.

### Contact Form
- Netlify Forms integrated (no backend, no Formspree). The form lives in `app/pages/contacto.vue`.

### Nuxt Content v3
- `@nuxt/content` v3 active. Collections defined in `content.config.ts`.
- 12 collections: `timeline_es`, `timeline_en`, `historia_es`, `historia_en`, `ciencia_articles`, `science_articles`, `team_es`, `team_en`, `press_es`, `press_en`, `science_es`, `science_en`.
- Timeline: `.yml` files in `content/es/timeline.yml` and `content/en/timeline.yml`.
- Story and science: `.md` files in `content/es/historia/`, `content/en/story/`, `content/es/ciencia/`, `content/en/science/`.
- Team data: `content/es/team.yml` and `content/en/team.yml` (3 groups: `coreTeam`, `medicalNetwork`, `integrativeSupport`).
- Press mentions: `content/es/press.yml` and `content/en/press.yml`.
- Science page data (treatments, papers, panel): `content/es/science.yml` and `content/en/science.yml`.
- Articles/chapters are linked between languages with `translationKey` in frontmatter.
- Language-specific slugs: ES `/ciencia/analisis-fgfr1`, EN `/en/science/fgfr1-analysis` — custom routes configured in `nuxt.config.ts` under `i18n.pages`.
- **Page collections** (`historia_es`, `historia_en`, `ciencia_articles`, `science_articles`) have their schemas extended with `defineRobotsSchema()` and `defineSitemapSchema()`. This enables per-file SEO control via frontmatter (e.g. `robots: noindex`, `sitemap: { priority: 0.8 }`).
- **Data collections** (timeline, team, press, science YML) do NOT have SEO schemas — they are not routed pages.
- If you modify `content.config.ts`, run `pnpm nuxt prepare` to regenerate types.

## Project Structure

```
app/
  app.vue                → Root app shell
  layouts/
    default.vue          → Default layout (nav + footer wrapper)
  pages/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BeyondTheProtocol/miriam-gonzalez-case](https://github.com/BeyondTheProtocol/miriam-gonzalez-case) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
