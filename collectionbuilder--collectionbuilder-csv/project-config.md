---
trigger: always_on
description: CollectionBuilder-CSV is a Jekyll-based framework for building digital collection and exhibit websites from a spreadsheet of metadata plus a folder of objects. It follows a **minimal computing** philosophy: static output, few dependencies, and configuration over code. The framework is **data-driven** — nearly all UI configuration comes from CSV files and YAML, not from editing HTML/Liquid templates. Understanding and following this architecture is critical.
---

# CollectionBuilder-CSV — AI Agent Instructions

CollectionBuilder-CSV is a Jekyll-based framework for building digital collection and exhibit websites from a spreadsheet of metadata plus a folder of objects. It follows a **minimal computing** philosophy: static output, few dependencies, and configuration over code. The framework is **data-driven** — nearly all UI configuration comes from CSV files and YAML, not from editing HTML/Liquid templates. Understanding and following this architecture is critical.

> **Companion files in this repo:** `CLAUDE.md` imports this file so Claude Code and Claude Cowork get the same rules. `HUMANS.md` is the human-facing guide to working with AI on this project. Keep all architecture rules here in `AGENTS.md` — the other two point back to it.

## Customization Priority Order

**Always check if a change can be made at a higher level before editing lower-level files:**

1. **`_config.yml`** — Site identity, org branding, and the `metadata:` pointer (filename of the active metadata CSV in `_data/`, no `.csv` extension)
2. **`_data/theme.yml`** — Visual behavior: navbar colors, map/timeline settings, typography, featured image, browse/search feature toggles
3. **`_data/config-*.csv`** — Controls what fields appear on item pages, browse cards, search index, map popups, nav links, and table columns
4. **`pages/about.md` and content pages** — Authored content using `_includes/feature/` includes
5. **`_sass/_custom.scss`** — CSS-level overrides only when the above layers cannot accomplish the goal

## Critical Rules — Never Violate These

**Navigation**
- Never edit `_includes/collection-nav.html` to add or remove links
- Add rows to `_data/config-nav.csv` instead
- The `dropdown_parent` column enables dropdown menus

**Item Page Metadata Fields**
- Never edit item layout HTML to show/hide fields
- Edit `_data/config-metadata.csv`: add/remove rows and set `display_name`, `browse_link`, and `external_link`

**Browse Page Cards**
- Fields on browse cards, facets, and sort options are configured in `_data/config-browse.csv`
- Do not touch `_layouts/browse.html` for field-level changes

**Custom CSS**
- Write all custom styles in `_sass/_custom.scss`
- Never modify `_base.scss`, `_pages.scss`, or `_theme-colors.scss`
- `_custom.scss` is `@use`d last in `assets/css/cb.scss` and wins specificity

**Metadata Pointer**
- `_config.yml` has `metadata: <filename>` with **no `.csv` extension**
- The collection is accessed as `site.data[site.metadata]` — bracket notation is required, not dot notation

**Item Pages Are Auto-Generated**
- `_plugins/cb_page_gen.rb` generates item pages from the metadata CSV at build time
- Never create `.html` files in `items/`
- The `display_template` column in the metadata CSV selects the layout
- Valid values: `image`, `video`, `pdf`, `audio`, `record`, `compound_object`, `panorama`, `multiple` (no `item/` prefix)

**Bootstrap 5**
- This project uses Bootstrap 5
- Use `ms-`/`me-` (not `ml-`/`mr-`), `float-start`/`float-end`, `data-bs-toggle=` (not `data-toggle=`)
- Use `d-flex`/`gap-*` utilities

**Production-Only Features**
- Analytics, Schema.org/OG meta tags, and `noindex` settings are wrapped in `{% if jekyll.environment == "production" %}`
- They will not appear during `bundle exec jekyll serve`

## Ask Before Doing These

Some changes are legitimate but destructive or hard to undo. Explain your plan and get the user's confirmation before:

- **Restructuring or reordering columns** in `_data/<metadata>.csv` (other config CSVs reference these column names)
- **Bulk-editing many item rows** at once, rather than a targeted change
- **Changing `display_template`** across the whole collection
- **Rewriting a config CSV's structure** (vs. adding or removing a single row)
- **Deleting rows** from any `config-*.csv` — confirm the field isn't referenced elsewhere first

## Data Files Quick Reference

| File | What it controls |
|------|-----------------|
| `_config.yml` | Site identity, `metadata:` CSV pointer, org info |
| `_data/theme.yml` | Visual settings, page features, map/timeline/browse config |
| `_data/config-metadata.csv` | Fields shown on item pages |
| `_data/config-browse.csv` | Browse card fields, facets, sort options |
| `_data/config-search.csv` | Lunr.js indexed fields and search result display |
| `_data/config-nav.csv` | Navbar and footer navigation links |
| `_data/config-map.csv` | Map popup field display |
| `_data/config-table.csv` | Data page table columns |
| `_data/config-theme-colors.csv` | Bootstrap color class overrides (primary, secondary, etc.) |
| `_data/<metadata>.csv` | The collection items; drives all visualizations |

## Feature Includes

For rich content in `about.md` and other Markdown pages, always use `_includes/feature/` includes — do not write raw HTML. Every feature is a drop-in Liquid include:

```liquid

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CollectionBuilder/collectionbuilder-csv](https://github.com/CollectionBuilder/collectionbuilder-csv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
