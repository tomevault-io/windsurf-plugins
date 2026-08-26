---
trigger: always_on
description: CollectionBuilder-Essay (**CB-Essay**) extends CollectionBuilder-CSV into a framework for long-form, illustrated scholarly writing — essays, monographs, and digital exhibits — built on top of a CSV-driven digital collection. It keeps CB-CSV's **minimal computing** philosophy (static output, few dependencies, configuration over code) and its **data-driven** architecture: nearly all UI configuration comes from CSV and YAML, not from editing HTML/Liquid templates. Following this architecture is cri
---

# CollectionBuilder-Essay — AI Agent Instructions

CollectionBuilder-Essay (**CB-Essay**) extends CollectionBuilder-CSV into a framework for long-form, illustrated scholarly writing — essays, monographs, and digital exhibits — built on top of a CSV-driven digital collection. It keeps CB-CSV's **minimal computing** philosophy (static output, few dependencies, configuration over code) and its **data-driven** architecture: nearly all UI configuration comes from CSV and YAML, not from editing HTML/Liquid templates. Following this architecture is critical.

CB-Essay is built **on top of CB-CSV**, so the entire CB-CSV ruleset below still applies. The essay layer adds essay content files, essay-specific includes, an expanded theming system, homepage styles, section navigation, and print/PDF output.

> **Companion files in this repo:** `CLAUDE.md` imports this file so Claude Code and Claude Cowork get the same rules. `HUMANS.md` is the human-facing guide to working with AI on this project. Keep all architecture rules here in `AGENTS.md` — the other two point back to it.

## Customization Priority Order

**Always check if a change can be made at a higher level before editing lower-level files:**

1. **`_config.yml`** — Site identity, org branding, and the `metadata:` pointer (filename of the active metadata CSV in `_data/`, no `.csv` extension)
2. **`_data/theme.yml`** — Visual behavior: **color theme, homepage image style, fonts, navigation toggles, print/PDF settings**, navbar, map/timeline, browse/search features
3. **`_data/config-*.csv`** — Controls what fields appear on item pages, browse cards, search index, map popups, nav links, and table columns
4. **`_essay/*.md`** — The essay/monograph content itself, authored in Markdown using essay feature includes
5. **`pages/about.md` and content pages** — Authored content using `_includes/feature/` includes
6. **`_sass/_custom.scss`** — CSS-level overrides only when the above layers cannot accomplish the goal

## Critical Rules — Never Violate These

These carry over from CB-CSV unchanged:

**Navigation** — Never edit `_includes/collection-nav.html`. Add rows to `_data/config-nav.csv` (the `dropdown_parent` column enables dropdowns).

**Item Page Metadata Fields** — Never edit item layout HTML. Edit `_data/config-metadata.csv` (`display_name`, `browse_link`, `external_link`).

**Browse Page Cards** — Configure browse fields, facets, and sort in `_data/config-browse.csv`, not `_layouts/browse.html`.

**Custom CSS** — Write all custom styles in `_sass/_custom.scss`. Never modify `_base.scss`, `_pages.scss`, `_theme-colors.scss`, `_color-tokens.scss`, `_essay.scss`, or `_print-paged.scss`. `_custom.scss` is `@use`d last and wins specificity.

**Metadata Pointer** — `_config.yml` has `metadata: <filename>` with **no `.csv` extension**. Access the collection as `site.data[site.metadata]` — bracket notation, not dot notation.

**Item Pages Are Auto-Generated** — `_plugins/cb_page_gen.rb` generates item pages from the metadata CSV at build time. Never create `.html` files in `items/`. The `display_template` column selects the layout. Valid values: `image`, `video`, `pdf`, `audio`, `record`, `compound_object`, `panorama`, `multiple` (no `item/` prefix).

**Bootstrap 5** — Use `ms-`/`me-` (not `ml-`/`mr-`), `float-start`/`float-end`, `data-bs-toggle=`, `d-flex`/`gap-*`.

**Production-Only Features** — Analytics, Schema.org/OG meta tags, and `noindex` are wrapped in `{% if jekyll.environment == "production" %}` and won't appear during `bundle exec jekyll serve`.

### Essay-Layer Rules — Also Never Violate

**Essay Content Lives in `_essay/`** — Each essay/chapter is a Markdown file in `_essay/` with YAML front matter (`title`, `order`, optional `byline`, `featured-image`). The **`order` field controls reading sequence**; the filename controls the URL. Don't hand-edit the essay layout templates (`_layouts/essay-content.html`, `home-essay.html`, etc.) to change content or ordering — use front matter.

**Use Essay Feature Includes, Not Raw HTML** — For rich essay content, use the essay includes in `_includes/essay/feature/` (and `_includes/essay/new-section.html`). Don't write raw HTML for blockquotes, asides, or galleries inside essays.

**Theme Settings Drive Appearance** — Color theme, homepage image style, fonts, and navigation toggles are all set in `_data/theme.yml`. Never edit `_sass/_color-tokens.scss` or `_essay.scss` to recolor or re-font the site — change `color-theme`, `base-font-family`, `display-font-family`, or `_data/config-theme-colors.csv` instead.

**Print/PDF Output** — Print and PDF are generated by Paged.js from `_data/theme.yml`'s `print:` block and `_sass/_print-paged.scss`. Configure via the `print:` keys; don't edit the paged stylesheet directly.

## Theming (CB-Essay's expanded system)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CollectionBuilder/cb-essay](https://github.com/CollectionBuilder/cb-essay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
