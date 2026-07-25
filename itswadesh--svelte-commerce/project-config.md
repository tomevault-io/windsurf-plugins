---
trigger: always_on
description: Always emit clean, SEO-friendly listing URLs. Do NOT link to
---

﻿# Link Conventions

Always emit clean, SEO-friendly listing URLs. Do NOT link to
`/products?search=<term>`; use the bare slug route instead: `/<term>`.

- `/products?search=pendant` → `/pendant`
- `/products?search=gold-ring` → `/gold-ring`

The bare `(www)/[slug]` route resolves the slug as a product listing (Meilisearch,
`categories: slug`), so `/pendant` is the canonical equivalent of a term/category search.
Apply this in `href`s, nav/menu links, category chips, search suggestions, and any generated
markup. This is a standing rule — do not ask about it again.

Keep real query params only when they carry non-term filters that the slug route reads from the
querystring (`price`, `tags`, `originCountry`, `keywords`, `page`) — e.g. `/pendant?page=2`.
The `products` route with its own routing/filter UI (the full catalogue page) is exempt; this rule
targets term/category shortcut links, not that page's internal state.

# Reskin rules (varni-reskin)

This repo gets re-themed per client using the varni-reskin pipeline in `tools/varni-reskin/`.
Project defaults live in `reskin.config.json` at the project root. Always let the tool read them
instead of asking the user for flags.

## Run Exactly One Store
Whenever the user asks to reskin a store, points at a static site folder, or drops a new folder
under `stores/`, run:

```bash
node ./tools/varni-reskin/bin/reskin.js auto --in "<single-store-path>"
```

If the user gives a parent folder such as `stores/`, stop and ask which single store to process.
Do not run batch mode by default. The user wants one store reskinned at a time.

`reskin.config.json` supplies `out`, `concurrency`, and `engine`. Do not ask the user for those
flags unless the requested store path is ambiguous or they explicitly ask for different settings.

The `/reskin <path>` slash command wraps this same single-store call.

## Switchable Theme Architecture
Multiple storefront themes must coexist. Do not overwrite the current theme as the only app design.
Add new themes as selectable implementations and resolve the active theme at runtime/config time.

- `.env` can select the active local/deployment theme with `PUBLIC_STOREFRONT_THEME=<theme-name>`.
- Admin/store settings should be allowed to override env when present. The resolver currently checks
  `store.theme`, `store.activeTheme`, `store.themeName`, `store.settings.theme`,
  `store.plugins.themeSettings.theme`, and `store.plugins.themeSettings.activeTheme`.
- The root layout exposes `data.theme` and adds `data-theme="<theme-name>"` to the app shell.
- Theme-specific homepage, nav, footer, component, and CSS work must be guarded by the active theme.
  A Sarab section must only render for the `sarab` theme; future themes should not inherit Sarab
  restaurant copy, images, icons, or layout by accident.
- Every new theme must include a cloned source header, navigation/mobile navigation, and footer
  layout, without copying the source brand identity. Treat header and footer as required sections,
  not optional shared chrome. Extract their source
  HTML/CSS, logo placement, menu structure, announcement/top bars, cart/account/search actions,
  social/payment links, newsletter/legal rows, responsive breakpoints, spacing, colors, borders,
  typography, and hover/focus states into that theme's `DESIGN.md` and implementation. Replace the
  source logo, brand name, domain name, email addresses, phone numbers, physical address, copyright
  text, and other unique identity marks with the target store/theme identity or a clearly different
  placeholder. Do not call a theme done while it still uses a previous theme's header or footer.
- Do not use exact real logos, copied logo marks, or lookalike logo artwork. Replace logo marks with
  the website name as a text wordmark. By default, rename the brand/site to `Website-$1`, where `$1`
  is the user-provided or inferred source brand/site identifier.
- Avoid copyright and plagiarism risk. Reference sites, screenshots, static templates, and competitor
  designs are inspiration and analysis inputs only. Absorb layout ideas, interaction patterns, and
  commerce requirements, then rebuild them inside this app's own theme identity and design system.
  Do not copy proprietary logos, brand names, domains, product names, campaign slogans, long-form
  copy, photography, illustrations, icon sets, reviews, policies, urgency tactics, or distinctive
  trade dress wholesale. Use original target-store content, licensed/local assets, API data, or
  clearly different placeholders.
- Homepage theme copy/assets must live in the theme layer, not inline in route files. Put hero copy,
  SEO title/image, section labels/headings/body copy, decorative image paths, gallery/testimonial/team
  entries, static story/history blocks, empty-state copy, and CTA labels in the active theme config.
  Route files should render `themeContent` plus API commerce data.
- Homepage data other than the actual product list must come from static theme content, not API page
  data. Hero banners, promo banners, newsletter/app-download banners, CTA copy, section headings,
  decorative images, trust badges, stats, testimonials, blog teasers, and theme-owned category/promo

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [itswadesh/svelte-commerce](https://github.com/itswadesh/svelte-commerce) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
