---
trigger: always_on
description: >
---


# AK Elementor Studio
### by [Abanoub Khalil](https://akstudio.me) — Senior WordPress & Webflow Developer

---

## RULE #1 — THE MOST IMPORTANT RULE IN THIS SKILL

```
HTML WIDGET = JAVASCRIPT <script> TAGS ONLY.

The html widget has exactly one valid use:
  → Embedding a <script> tag for JavaScript behavior

That is ALL. Nothing else ever goes in the html widget.
Not headings. Not text. Not badges. Not nav links.
Not cards. Not layouts. Not pricing. Not timelines.
Not icons. Not stats. Not ANY visual content.

Every visual element has a native Elementor widget. Use it.
```

---

## RULE #2 — ABSOLUTE JSON RULES

```
1. NO COMMENTS — /* */ and // both break JSON parsing
   ❌ { /* hero */ "id": "a1b2c3d4" }    ✅ { "id": "a1b2c3d4" }

2. NO TRAILING COMMAS
   ❌ { "a": 1, "b": 2, }               ✅ { "a": 1, "b": 2 }

3. IDs = UNIQUE 8-CHAR HEX ONLY
   ❌ "id": "hero_section"              ✅ "id": "a1b2c3d4"

4. EVERY ELEMENT MUST HAVE "elements": []
   ❌ { "widgetType": "heading", "settings": {} }
   ✅ { "widgetType": "heading", "settings": {}, "elements": [] }

5. isInner RULE
   Direct children of "content": []        → "isInner": false
   Containers nested inside containers      → "isInner": true
   Widgets (inside any container)           → "isInner": false

6. HEX COLORS ONLY — never color names
   ❌ "background_color": "white"       ✅ "background_color": "#FFFFFF"
```

---

## STEP 0 — Pre-Flight Interview

Ask all in ONE message. Never skip.

```
👋 Quick details before I start:

1. SITEMAP — Pages? (I generate one .json per page)

2. ELEMENTOR VERSION — Free or Pro?
   Pro: Theme Builder header/footer, custom CSS, sticky, scroll effects
   Free: header/footer = regular sections inside each page

3. CUSTOM POST TYPES — Any CPTs?
   (Services, Portfolio, Plugins, Team, Testimonials → I'll generate ACF JSON)

4. OUTPUT FORMAT
   a) Individual .json files — import one-by-one via Templates → Saved Templates (always works, Free + Pro)
   b) Elementor native export ZIP — import via Elementor → Tools → Import Kit (no extra plugin needed)
   c) Envato Template Kit ZIP — requires "Template Kit Import" plugin installed on WordPress
   d) Individual .json + ACF field group files

5. BRAND — Hex color codes + Google Fonts?
   (Or I extract from the design)
```

---

## STEP 1 — Input Detection

| Input | Pipeline |
|---|---|
| HTML/CSS/JS | Code Pipeline |
| PNG/JPG/WebP/screenshot/mockup/Figma | Visual Pipeline |
| PDF | PDF Pipeline |
| Words only | Description Pipeline |
| Existing Elementor JSON | Edit Pipeline |
| Mixed | Both pipelines, merged |

---

## COMPLETE WIDGET REFERENCE — Native Widgets for Every Element

### Text & Numbers
| Element | Widget |
|---|---|
| h1 / h2 / h3 / h4 / h5 / h6 | `heading` |
| Paragraph / body text | `text-editor` |
| Animated counter (500+, 10K) | `counter` |
| Progress / skill bar | `progress` |

### Actions
| Element | Widget |
|---|---|
| Button / CTA / link-as-button | `button` |
| FAQ / collapsible rows | `accordion` |
| Tab navigation | `tabs` |
| Alert / info banner | `alert` |

### Media
| Element | Widget |
|---|---|
| Photo / image | `image` |
| YouTube / Vimeo / video | `video` |
| Image grid | `image-gallery` |
| Image slider / carousel | `image-carousel` |

### Cards & Lists
| Element | Widget |
|---|---|
| Icon + title + description | `icon-box` |
| Bulleted list with icons | `icon-list` |
| Photo + title + text | `image-box` |
| Customer quote | `testimonial` |

### Layout Helpers
| Element | Widget |
|---|---|
| Horizontal line / separator | `divider` |
| Vertical blank space | `spacer` |

### Utilities
| Element | Widget |
|---|---|
| Social media icons | `social-icons` |
| Map embed | `google-maps` |
| Plugin/form shortcode | `shortcode` |

---

## SOLVING COMPLEX ELEMENTS WITHOUT html WIDGET

### Navigation Bar
```
Structure: flex container (row, space-between)
  └── image widget          ← logo
  └── icon-list widget      ← nav links (set icon to "none", each item = a page link)
      OR multiple button widgets (size: xs, style: link/ghost, one per nav item)
  └── button widget         ← CTA button
```

### Badge / Status Pill / Tag
```
Option A: button widget
  text: "FULL-TIME", size: "xs", border_radius: 100px
  button_text_color: "#00D4C8", button_background_color: "rgba(0,212,200,0.1)"
  no link href (or link to "#")

Option B: alert widget
  type: "success" / "info" / "warning"
  alert_title: "WordPress Developer"
  (style with custom CSS if Pro)
```

### Career Timeline / Work History Entry
```
Structure: inner container (column, border-left: 2px solid accent)
  └── inner container (row, gap: 8px)   ← date + status row
      └── text-editor widget            ← "OCT 2024 — PRESENT"
      └── button widget (xs, pill)      ← "FULL-TIME" badge
  └── heading widget (h3/h4)            ← "Senior WordPress & Webflow Developer"
  └── text-editor widget                ← "InVitro Capital — Cairo, Egypt"
  └── text-editor widget                ← description paragraph
  └── inner container (row, wrap)       ← tech tags row
      └── button widget (xs, ghost)     ← "WordPress"
      └── button widget (xs, ghost)     ← "Webflow"
      └── button widget (xs, ghost)     ← "CMS"
```

### Contact Info Panel
```
Structure: styled container (background: #141628, border-radius: 12px, padding: 32px)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abanoubkhaliil/Ak-Elementor-Studio](https://github.com/abanoubkhaliil/Ak-Elementor-Studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
