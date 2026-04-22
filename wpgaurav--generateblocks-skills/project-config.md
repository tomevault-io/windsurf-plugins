---
trigger: always_on
description: Universal instructions for all LLM assistants working with GenerateBlocks skills.
---

# AGENTS.md

Universal instructions for all LLM assistants working with GenerateBlocks skills.

## GenerateBlocks V2 Block Types

GenerateBlocks V2 uses generic element-based blocks. **CRITICAL: Use the correct block names.**

| Block Type | Correct Name | NOT These |
|------------|--------------|-----------|
| Containers | `generateblocks/element` | ❌ `/container`, `/grid` |
| Text/Buttons | `generateblocks/text` | ❌ `/headline`, `/button` |
| Images | `generateblocks/media` | ❌ `/image` |
| SVG Icons | `generateblocks/shape` | - |

## Required Class Naming

Classes MUST follow this pattern:

| Block | Class Pattern | Example |
|-------|---------------|---------|
| Element | `gb-element-{uniqueId} gb-element` | `gb-element-hero001 gb-element` |
| Text | `gb-text gb-text-{uniqueId}` | `gb-text gb-text-hero002` |
| Media | `gb-media gb-media-{uniqueId}` | `gb-media gb-media-hero003` |
| Shape | `gb-shape gb-shape-{uniqueId}` | `gb-shape gb-shape-hero004` |

## Block Structure Examples

### Element Block (Container)

```html
<!-- wp:generateblocks/element {"uniqueId":"hero001","className":"gb-element-hero001 gb-element","tagName":"section","styles":{"paddingTop":"4rem","paddingBottom":"4rem"},"css":".gb-element-hero001{padding:4rem 0}"} -->
<section class="gb-element-hero001 gb-element">
    <!-- Inner blocks -->
</section>
<!-- /wp:generateblocks/element -->
```

### Text Block

```html
<!-- wp:generateblocks/text {"uniqueId":"hero002","tagName":"h1","styles":{"fontSize":"3rem","fontWeight":"800","color":"#0a0a0a"},"css":".gb-text-hero002{font-size:3rem;font-weight:800;color:#0a0a0a}"} -->
<h1 class="gb-text gb-text-hero002">Welcome to Our Site</h1>
<!-- /wp:generateblocks/text -->
```

### Link/Button (also uses Text block)

```html
<!-- wp:generateblocks/text {"uniqueId":"hero003","tagName":"a","styles":{"display":"inline-block","padding":"1rem 2rem","backgroundColor":"#c0392b","color":"#ffffff","borderRadius":"8px"},"css":".gb-text-hero003{background-color:#c0392b;border-radius:8px;color:#fff;display:inline-block;padding:1rem 2rem;text-decoration:none}"} -->
<a class="gb-text gb-text-hero003">Get Started</a>
<!-- /wp:generateblocks/text -->
```

### Media Block (Image)

```html
<!-- wp:generateblocks/media {"uniqueId":"hero004","mediaType":"image","htmlAttributes":{"src":"https://example.com/image.jpg","alt":"Description","loading":"lazy","width":"800","height":"600"},"styles":{"width":"100%","borderRadius":"1rem"},"css":".gb-media-hero004{width:100%;border-radius:1rem}"} -->
<img class="gb-media gb-media-hero004" src="https://example.com/image.jpg" alt="Description" loading="lazy" width="800" height="600" />
<!-- /wp:generateblocks/media -->
```

### Shape Block (SVG Icon)

```html
<!-- wp:generateblocks/shape {"uniqueId":"hero005","styles":{"width":"24px","height":"24px","fill":"currentColor"},"css":".gb-shape-hero005{width:24px;height:24px;fill:currentColor}"} -->
<svg class="gb-shape gb-shape-hero005" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor">
    <path d="M12 2L2 7l10 5 10-5-10-5zM2 17l10 5 10-5M2 12l10 5 10-5"/>
</svg>
<!-- /wp:generateblocks/shape -->
```

## Key Attributes

| Attribute | Type | Purpose |
|-----------|------|---------|
| `uniqueId` | string | Required. Used for CSS class targeting. Format: `section###` or `section###letter` |
| `tagName` | string | HTML element type (div, section, h1, p, a, button, span, etc.) |
| `styles` | object | Basic CSS properties as JSON (camelCase keys) |
| `css` | string | Base styles (alphabetically sorted, minified). Exceptions: pseudo-elements, media queries, animations, parent hover targeting children |
| `htmlAttributes` | object | Plain object of HTML attrs: `{"href":"url","target":"_blank"}`. NOT array format |

## Unique ID Convention

Format: `{section}{number}{letter}`

- **Section**: 3-4 characters describing the section (hero, serv, card, blog, faq)
- **Number**: 001-999, sequential within the section
- **Letter**: Optional a-z for nested elements

Examples:
- `hero001` - Main hero container
- `hero001a` - Nested element within hero
- `card012b` - Second nested element in card 12

## CSS Rules

1. **`styles` attribute**: Use for basic properties (padding, margin, flex, grid, colors, typography)
2. **`css` attribute**: Base styles (alphabetically sorted, minified). Exceptions: pseudo-elements, media queries, animations, parent hover targeting children. **No hover states or transitions** (plugin generates from `styles`)
3. **Always minify CSS** in the `css` attribute (no line breaks)
4. **Target classes** use the uniqueId: `.gb-element-hero001`, `.gb-text-hero002`
5. **Icon containers need `line-height: 1`** - Elements presenting icons must have `lineHeight: "1"`
6. **Lists use `core/list` with `.list` class** - Native WordPress list block with `className: "list"`
7. **Use `--gb-container-width`** - For inner container width; add `align: "full"` to parent section

## Additional Formatting Rules

1. **Full absolute URLs required** - Never use relative paths (e.g., `https://example.com/image.jpg` not `/image.jpg`)
2. **No spaces in CSS functions** - `clamp(3rem,8vw,5rem)` not `clamp(3rem, 8vw, 5rem)`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wpgaurav/generateblocks-skills](https://github.com/wpgaurav/generateblocks-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
