---
trigger: always_on
description: 🚨 MANDATORY: YOU MUST CALL "learn_shopify_api" ONCE WHEN WORKING WITH LIQUID THEMES.
---

🚨 MANDATORY: YOU MUST CALL "learn_shopify_api" ONCE WHEN WORKING WITH LIQUID THEMES.

## Theme Architecture

**Key principles: focus on generating snippets, blocks, and sections; users may create templates using the theme editor**

### Directory structure

```
.
├── assets          # Stores static assets (CSS, JS, images, fonts, etc.)
├── blocks          # Reusable, nestable, customizable components
├── config          # Global theme settings and customization options
├── layout          # Top-level wrappers for pages (layout templates)
├── locales         # Translation files for theme internationalization
├── sections        # Modular full-width page components
├── snippets        # Reusable Liquid code or HTML fragments
└── templates       # Templates combining sections and blocks to define page structures
```

#### `sections`

- Sections are `.liquid` files that allow you to create reusable modules that can be customized by merchants
- Sections can include blocks which allow merchants to add, remove, and reorder content within a section
- Sections are made customizable by including the required `{% schema %}` tag that exposes settings in the theme editor via a JSON object. Validate that JSON object using the `schemas/section.json` JSON schema
- Examples of sections: hero banners, product grids, testimonials, featured collections

#### `blocks`

- Blocks are `.liquid` files that allow you to create reusable small components that can be customized by merchants (they don't need to fit the full-width of the page)
- Blocks are ideal for logic that needs to be reused and also edited in the theme editor by merchants
- Blocks can include other nested blocks which allow merchants to add, remove, and reorder content within a block too
- Blocks are made customizable by including the required `{% schema %}` tag that exposes settings in the theme editor via a JSON object. Validate that JSON object using the `schemas/theme_block.json` JSON schema
- Blocks must have the `{% doc %}` tag as the header if you directly/staticly render them in other file via `{% content_for 'block', id: '42', type: 'block_name' %}`
- Examples of blocks: individual testimonials, slides in a carousel, feature items

#### `snippets`

- Snippets are reusable code fragments rendered in blocks, sections, and layouts files via the `render` tag
- Snippets are ideal for logic that needs to be reused but not directly edited in the theme editor by merchants
- Snippets accept parameters when rendered for dynamic behavior
- Snippets must have the `{% doc %}` tag as the header
- Examples of sections: buttons, meta-tags, css-variables, and form elements

#### `layout`

- Defines the overall HTML structure of the site, including `<head>` and `<body>`, and wraps other templates to provide a consistent frame
- Contains repeated global elements like navigation, cart drawer, footer, and usually includes CSS/JS assets and meta tags
- Must include `{{ content_for_header }}` to inject Shopify scripts in the `<head>` and `{{ content_for_layout }}` to render the page content

#### `config`

- `config/settings_schema.json` is a JSON file that defines schema for global theme settings. Validate the shape shape of this JSON file using the `schemas/theme_settings.json` JSON schema
- `config/settings_data.json` is JSON file that holds the data for the settings defined by `config/settings_schema.json`

#### `assets`

- Contains static files like CSS, JavaScript, and images—including compiled and optimized assets—referenced in templates via the `asset_url` filter
- Keep it here only `critical.css` and static files necessary for every page, otherwise prefer the usage of the `{% stylesheet %}` and `{% javascript %}` tags

#### `locales`

- Stores translation files organized by language code (e.g., `en.default.json`, `fr.json`) to localize all user-facing theme content and editor strings
- Enables multi-language support by providing translations accessible via filters like `{{ 'key' | t }}` in Liquid for proper internationalization
- Validate `locales` JSON files using the `schemas/translations.json` JSON schema

#### `templates`

- JSON file that define the structure, ordering, and which sections and blocks appear on each page type, allowing merchants to customize layouts without code changes

### CSS & JavaScript

- Write CSS and JavaScript per components using the `{% stylesheet %}` and `{% javascript %}` tags
- Note: `{% stylesheet %}` and `{% javascript %}` are only supported in `snippets/`, `blocks/`, and `sections/`

### LiquidDoc

Snippets and blocks (when blocks are statically rendered) must include the LiquidDoc header that documents the purpose of the file and required parameters. Example:

```liquid
{% doc %}
  Renders a responsive image that might be wrapped in a link.

  @param {image} image - The image to be rendered
  @param {string} [url] - An optional destination URL for the image

  @example
  {% render 'image', image: product.featured_image %}
{% enddoc %}

<a href="{{ url | default: '#' }}">{{ image | image_url: width: 200, height: 200 | image_tag }}</a>
```

## The `{% schema %}` tag on blocks and sections

**Key principles: follow the "Good practices" and "Validate the `{% schema %}` content" using JSON schemas**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shopify/theme-liquid-docs](https://github.com/Shopify/theme-liquid-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
