---
trigger: always_on
description: FlowStory is an AI-agent-first framework for creating animated, step-by-step architecture flow diagrams from declarative JSON. You define nodes, flows, tooltips, and inspector mutations in a single `diagram.json` file; the engine handles canvas rendering, dot animation, and interactive playback.
---

# CLAUDE.md

## What FlowStory Is

FlowStory is an AI-agent-first framework for creating animated, step-by-step architecture flow diagrams from declarative JSON. You define nodes, flows, tooltips, and inspector mutations in a single `diagram.json` file; the engine handles canvas rendering, dot animation, and interactive playback.

## JSON Schema Reference

A FlowStory diagram is a single JSON object with these top-level keys:

### `meta` (required)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | string | yes | Diagram title displayed at top center |
| `author` | string | no | Author name (metadata only) |
| `branding` | object | no | `{ logo: "path.svg", title: "Company" }` shown top-left |

### `canvas` (optional)

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `width` | number | 1250 | Logical canvas width in design units |
| `height` | number | 1050 | Logical canvas height in design units |

### `nodes` (required)

Object keyed by node ID string. Each node:

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `x` | number | -- | X position (top-left origin) |
| `y` | number | -- | Y position |
| `w` | number | -- | Width |
| `h` | number | -- | Height |
| `label` | string | -- | Display text |
| `type` | string | `"box"` | One of: `"box"`, `"icon"`, `"container"`, `"boundary"`, `"plugin"` |
| `sublabel` | string | -- | Smaller text below label |
| `color` | hex string | `"#58a6ff"` | Node border/accent color |
| `fontSize` | number | 16 | Label font size |
| `inner` | boolean | false | Visual hint that node is inside a container |
| `soon` | boolean | false | Adds "COMING SOON" badge |
| `sections` | array | -- | Container-only: plugin section regions (see below) |
| `stackCount` | number | -- | Number of stacked shadow layers behind the node |
| `stackOffset` | object | `{ dx: -8, dy: -5 }` | Per-layer offset for stack shadows |
| `subBoundaries` | array | -- | Boundary-only: inset dashed sub-regions |
| `labelAlign` | string | `"right"` | Boundary-only: `"left"` or `"right"` |
| `labelColor` | hex string | -- | Boundary-only: override label color |

**Sections** (on container nodes):
```json
"sections": [
  { "label": "Request Plugins ->", "y": 300, "height": 260, "color": "#58a6ff" },
  { "label": "<- Response Plugins", "y": 623, "height": 105, "color": "#3fb950" }
]
```
Each section: `{ x?, y, w?, h or height, color, label, labelX?, labelY? }`. Position is in logical coords. If `x` and `w` are omitted, the section fills the container width.

**SubBoundaries** (on boundary nodes):
```json
"subBoundaries": [
  { "x": 235, "y": 933, "w": 230, "h": 82, "color": "#d2a8ff44" }
]
```

### `tooltips` (optional)

Object keyed by node ID. Each tooltip is shown when the user clicks a node:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | string | yes | Overlay card title |
| `description` | string | no | Description paragraph |
| `details` | array | no | Array of `[key, value]` pairs displayed as a table |
| `links` | array | no | Array of `[text, url]` pairs displayed as buttons |
| `logo` | string | no | Path to logo image shown above title |

Example:
```json
"tooltips": {
  "server": {
    "title": "API Server",
    "description": "Handles all REST API requests.",
    "details": [
      ["Port", "8080"],
      ["Protocol", "HTTP/2"]
    ],
    "links": [
      ["Docs", "https://example.com/docs"]
    ]
  }
}
```

### `flows` (required)

Object keyed by flow ID. Each flow:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `label` | string | yes | Display name in the flow selector dropdown |
| `steps` | array | yes | Ordered array of step objects |

**Arrow step** -- animates a dot traveling between nodes, then draws a permanent arrow:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `text` | string | yes | Step description shown in the steps panel |
| `mode` | `"arrow"` | yes | Step type |
| `from` | string | yes | Source node ID |
| `to` | string | yes | Target node ID |
| `color` | hex string | yes | Arrow color |
| `num` | number | yes | Badge number displayed at arrow midpoint |
| `glow` | string | no | Container node ID to glow when arrow arrives |
| `fromRight` | boolean | no | Attach arrow from right edge of source |
| `fromLeft` | boolean | no | Attach from left edge |
| `fromTop` | boolean | no | Attach from top edge |
| `fromBottom` | boolean | no | Attach from bottom edge |
| `toRight` | boolean | no | Attach arrow to right edge of target |
| `toLeft` | boolean | no | Attach to left edge |
| `toTop` | boolean | no | Attach to top edge |
| `toBottom` | boolean | no | Attach to bottom edge |
| `fromXOff` | number | no | Horizontal offset on source edge |
| `toXOff` | number | no | Horizontal offset on target edge |
| `yOff` | number | no | Vertical offset for edge attachment |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [noyitz/flowstory](https://github.com/noyitz/flowstory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
