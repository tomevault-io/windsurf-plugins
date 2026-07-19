---
trigger: always_on
description: Guidance for working in this repository.
---

# CLAUDE.md

Guidance for working in this repository.

## What this repo is

**WooCommerce in Etch (Woo4Etch)** — an open-source (MIT) *documentation + template library*, plus a small companion WordPress plugin, for building WooCommerce shops in [Etch](https://etchwp.com) without the WooCommerce Gutenberg blocks.

This is **not a runnable application**. There is no build step or dev server, and the deliverables are Markdown docs and copy-ready snippets. "Running" anything means installing the `plugin/woo4etch/` plugin into a real WordPress + WooCommerce + Etch site and pasting template markup into Etch.

There *is* a fast, service-free test layer: `php tests/php/run.php` checks version-marker sync, the shortcode catalog, and the layout DSL invariants under plain PHP-CLI (no WordPress). It runs on every PR via `.github/workflows/test.yml` alongside a PHP 8.1→8.5 lint matrix (8.1 is Etch's own floor). See `tests/php/README.md`. The copy/paste artifacts in `templates/etch-copy/*.json` are generated from the layout definitions via `php tools/generate-etch-copy.php` (now WordPress-free too) and CI fails if they drift.

## Layout

| Path | What it is |
|---|---|
| `README.md` | Entry point + template index |
| `WooCommerce-in-Etch-Knowledgebase.md` | Long-form research notes (the "why") |
| `templates/00-README.md` | Conventions + **Shared foundations** (theme support, wrappers, quantity hooks) |
| `templates/01..09-*.md` | One template per Woo area (single product, cart, checkout, etc.) |
| `templates/10-*.md` | **Read first** — Etch context: `{this.*}` (Single template) vs `{item.*}` (inside loops) |
| `templates/11..14-*.md` | Conditional tags, Store/REST API, snippets, visual hook guides |
| `templates/15-woo4etch-plugin.md` | Plugin shortcode reference (`[do_action]` etc.) |
| `templates/functions-snippets.md` | All PHP snippets consolidated |
| `plugin/woo4etch/` | The actual WordPress plugin (PHP) |
| `plugin/woo4etch/includes/customizations.php` | **Empty by design** — where users paste hook/filter snippets |

## The core mental model (critical — most user confusion lives here)

1. **The layout is hand-written Etch HTML.** Add-to-cart forms, prices, quantity inputs — these are literal markup the user pastes into an Etch Single template, with Dynamic Keys like `{this.id}`. WooCommerce does **not** generate this markup in this approach (no Woo blocks, no `[product_page]` shortcode for the layout).

2. **Hooks are optional injection points, not content generators.** In the template `.md` files, lines like `<!-- Hook: woocommerce_before_add_to_cart_form -->` are just *placeholders showing where a hook would fire*. They render nothing on their own. To actually fire a hook in Etch you place a `[do_action hook="..."]` shortcode (from the Woo4Etch plugin) at that spot, and something must be hooked onto it (a plugin, or a snippet in `customizations.php`).

3. **`add_action(...)` in `customizations.php` only adds *extras*** (trust badges, ± buttons) *around* the form — it never produces the add-to-cart button itself. The button comes from step 1's markup.

4. **`{this.*}` vs `{item.*}`**: `this` = current Single template context; `item` = current loop iteration inside `{#loop … as item}`. Mixing them up is the #1 cause of empty fields. See `templates/10-etch-context-and-templates.md`.

5. The only *required* shared-foundation snippet is `add_theme_support('woocommerce')`. Everything else under "Shared foundations" (disabling Woo CSS, replacing content wrappers, quantity-button hooks) is optional.

## Conventions when editing

- Every template `.md` follows the fixed 8-section structure: When to use → Preparation → Etch HTML → Required classes/attributes → Hooks used → PHP layer → Common mistakes → Test checklist. Keep it.
- Tone is neutral reference documentation. Match it.
- Required Woo classes/attributes (`form.cart`, `name="quantity"`, `name="add-to-cart"`, `value="{this.id}"`, `.single_add_to_cart_button`) must never be dropped from example markup — Woo's server logic keys off them.
- PHP snippets target `customizations.php`, not a theme `functions.php` or a second plugin (unless the user prefers the theme).
- The template index appears in three places (`README.md`, `templates/00-README.md`, and as files). Keep them in sync and in numeric order.

## Verified Etch capabilities (verified against the Etch plugin source)

Etch is server-rendered FSE blocks. Ground truth confirmed against source — use these, not assumptions:

- **`{this.meta._price}` works.** `get_post_meta()` is exposed under a generic `meta` object and single-value meta is flattened to a string. So all `{this.meta._*}` keys in the docs are correct.
- **`{this.gallery_images}`** is a ready-to-loop array of image objects (`id, url, alt, title, caption, description, filename, sizes, srcset, width, height, filesize, mime_type`), added by Etch's WooCommerce integration (commit ETC-800) for `product` posts. **Available in Etch 1.4.20+ (released); not in 1.4.19.** Loop it in Etch with a loop whose target is `this.gallery_images`. Verified live on 1.4.20. It **excludes the featured image by design** — render the featured image separately, then loop the gallery. The raw `_product_image_gallery` meta is just a CSV of IDs and is not usable directly.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tobiashaas/woo4etch](https://github.com/tobiashaas/woo4etch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
