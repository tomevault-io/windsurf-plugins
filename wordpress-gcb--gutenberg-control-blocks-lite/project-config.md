---
trigger: always_on
description: Each block has a tiny PHP/JSON schema in the theme and one React component
---

# Authoring blocks against GCB Lite

Each block has a tiny PHP/JSON schema in the theme and one React component
in your Next.js (or Astro, or any HTTP-SSR) frontend. The same component
renders both the editor preview and the public site — WordPress hits one
route on your frontend to get the HTML when an author edits, and your
frontend renders the same component for visitors.

Two ways to render a block:

1. **PHP `render.php`** — standard WP block. The plugin auto-wires
   `render: file:./render.php` if it exists.
2. **React component on your frontend** — drop the `render.php`. The plugin
   asks your frontend to render via its `/wordpress/render/[block]` route.

A block can use either; the plugin picks based on whether `render.php`
exists. See [README.md](./README.md) for the overall architecture.

## Where blocks live

```
themes/{active-theme}/blocks/{block-name}/
├── block.json          # Standard WP block metadata
├── block.fields.json   # GCB controls config (optional)
├── render.php          # Server-side render (optional — see "Rendering with a React component")
└── style.css           # Frontend + editor styles (optional)
```

Block name = directory name. WP registers it as `gcb/{block-name}`.

## What you write

### `block.json`

Standard WP block metadata. Nothing GCB-specific in here:

```json
{
  "$schema": "https://schemas.wp.org/trunk/block.json",
  "apiVersion": 3,
  "name": "gcb/team-grid",
  "title": "Team Grid",
  "category": "widgets",
  "icon": "groups",
  "textdomain": "gcb",
  "attributes": {},
  "supports": {}
}
```

Rules:
- `supports` must be `{}` or `[]`.
- `attributes` should be `{}` — they're auto-generated from `block.fields.json`.
- `render` is auto-wired by the plugin if `render.php` exists; no need to set it.

### `block.fields.json`

GCB-specific. Declares Inspector controls; the plugin auto-generates the
WP attribute defs from these.

```json
{
  "controls": [
    {
      "id": "panel_main",
      "type": "group",
      "label": "Main",
      "controlsGroup": "settings"
    },
    {
      "id": "ctrl_heading",
      "type": "text",
      "label": "Heading",
      "attributeKey": "heading",
      "controlsGroup": "settings",
      "parentPanelId": "panel_main"
    }
  ]
}
```

For controls: `id` is unique within the block; `attributeKey` is required for non-`group` types.

The schema file is at `schemas/gcb.schema.json` — point your editor at it via `$schema` in your `block.fields.json` for autocomplete (optional).

### IMPORTANT: picking the right control type

Get this wrong and the attribute ends up the wrong shape (e.g. array instead
of string), the component reads `undefined`, and the block looks broken.
Pick by **the shape of the saved value**, not by what looks nicest in the UI:

| You want…                                          | Control          | Stored type | Notes                                                  |
|----------------------------------------------------|------------------|-------------|--------------------------------------------------------|
| Free-text single line                              | `text`           | string      |                                                        |
| Free-text multi-line                               | `textarea`       | string      |                                                        |
| Rich text (bold, links, lists)                     | `wysiwyg`        | string      | HTML. Often better as InnerBlocks — see below.         |
| **On/off toggle**                                  | `toggle`         | boolean     | A real switch. Not a checkbox.                         |
| One-of-N from a small fixed set (≤4, visual)       | `toggle-group`   | string      | Segmented control. Single-value. Use for left/right.   |
| One-of-N from a larger set                         | `select`         | string      | Dropdown.                                              |
| One-of-N visible all at once                       | `radio`          | string      | Stacked radio buttons.                                 |
| **Many-of-N**                                      | `checkbox-group` | array       | Multi-select. `button-group` is an alias for this.     |
| Single boolean question                            | `checkbox`       | boolean     | When a `toggle` switch is the wrong affordance.        |
| Number                                             | `number`         | number      | Free-typed.                                            |
| Number on a fixed scale                            | `range`          | number      | Slider with min/max/step.                              |
| Colour                                             | `color`          | string      | Hex / theme palette.                                   |
| Single image                                       | `image`          | object      | `{ id, url, alt, width, height, ... }`.                |
| Gallery of images                                  | `gallery`        | array       | `[{ id, url, alt, ... }, ...]`.                        |
| Other media (PDF, video, etc.)                     | `file`           | object      |                                                        |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wordpress-gcb/gutenberg-control-blocks-lite](https://github.com/wordpress-gcb/gutenberg-control-blocks-lite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
