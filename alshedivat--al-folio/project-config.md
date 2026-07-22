---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@AGENTS.md

The import above (`AGENTS.md`, which itself defers to `.github/copilot-instructions.md` and `docs/BOUNDARIES.md`) is the canonical short entry point: ownership boundaries, the validated command set, and PR-routing rules. Keep `AGENTS.md` short and ecosystem-neutral — put Claude-specific or longer-form guidance here instead. Everything below is the cross-repo "big picture" that those files assume but don't spell out.

## What this repo is

`al-folio` v1.x is a **thin Jekyll starter**, not a theme. It owns only: starter wiring (`Gemfile`, `_config.yml`, `_data/featured_plugins.yml`), example content (`_pages`, `_posts`, `_projects`, `_news`, `_teachings`, `_books`, `_bibliography`), docs (`docs/`), cross-gem integration tests (`test/integration_*.sh`), and visual/parity tests (`test/visual/`). **All runtime, layouts, includes, Sass, tags, filters, and feature JS live in versioned gems**, published independently on RubyGems. `docs/BOUNDARIES.md` is the authoritative area→gem ownership table.

The biggest recurring mistake is editing runtime here. If a change is layout/include/tag/filter/feature-behavior, it belongs in the owning gem (see routing below), not in this repo.

## The plugin ecosystem (read this before routing any change)

The `al-*` / `al_folio_*` gems are developed as **sibling repos on disk** at `~/Documents/dev/al-org/<repo>` (repo dir uses hyphens, e.g. `al-folio-core`; gem/plugin id uses underscores, e.g. `al_folio_core`). To test a gem fix against this site, point the `Gemfile` at it: `gem "al_folio_core", path: "../al-folio-core"` (or `git:`/`branch:`), then `bundle install`.

**`al_folio_core` is the hub.** `_config.yml` sets `theme: al_folio_core`; the gem ships every base `_layouts/*.liquid` and `_includes/*.liquid`, the base theme JS/CSS, and the `details`/`file_exists` tags + `hideCustomBibtex`/`remove_accents` filters. Crucially, its `_includes/plugins/*.liquid` are **thin wrappers that call custom Liquid tags defined by sibling gems**. So a feature renders only when _both_ (a) its gem is present in the plugin list, and (b) the relevant flag is on. The wrapper→tag→gem delegation map:

| Wrapper / call site       | Tag                                                 | Gem                                                                            |
| ------------------------- | --------------------------------------------------- | ------------------------------------------------------------------------------ |
| search assets             | `al_search_assets`                                  | `al_search` (Cmd-K ninja-keys palette; index built at build time from content) |
| comments                  | `al_comments`                                       | `al_comments` (Giscus + Disqus, front-matter gated)                            |
| cookie banner             | `al_cookie_styles` / `al_cookie_scripts`            | `al_cookie` (consent-mode gating of analytics)                                 |
| icon `<link>`s            | `al_icons_styles`                                   | `al_icons` (FontAwesome/Academicons/Scholar Icons from CDN)                    |
| analytics                 | `al_analytics_scripts`                              | `al_analytics` (GA/Cronitor/Pirsch/OpenPanel)                                  |
| math                      | `al_math_styles` / `al_math_scripts`                | `al_math` (MathJax, pseudocode.js, TikZJax)                                    |
| charts                    | `al_charts_scripts`                                 | `al_charts` (Mermaid/Chart.js/ECharts/Plotly/Vega/Leaflet/diff2html)           |
| image tools               | `al_img_tools_styles` / `al_img_tools_scripts`      | `al_img_tools` (zoom, lightbox, sliders, galleries)                            |
| newsletter                | `al_newsletter_form` / `al_newsletter_scripts`      | `al_newsletter` (Loops.so signup)                                              |
| `layout: cv`              | `al_folio_cv_render`                                | `al_folio_cv` (RenderCV YAML + JSONResume)                                     |
| `layout: distill`         | `al_folio_distill_render`                           | `al_folio_distill` (vendored, hash-pinned distillpub runtime)                  |
| citation badges           | `google_scholar_citations` / `inspirehep_citations` | `al_citations`                                                                 |
| external posts            | (generator, no tag)                                 | `al_ext_posts` (RSS/URL ingestion → synthetic posts)                           |
| legacy Bootstrap behavior | (opt-in assets)                                     | `al_folio_bootstrap_compat`                                                    |
| upgrade/audit CLI         | `bundle exec al-folio …`                            | `al_folio_upgrade`                                                             |

Architectural facts that span repos:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alshedivat/al-folio](https://github.com/alshedivat/al-folio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
