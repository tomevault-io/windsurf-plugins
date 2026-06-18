---
trigger: always_on
description: Generate clean SVG diagrams (flowchart, tree, mindmap, architecture, sequence, quadrant, gantt, state machine, ER, timeline, swimlane, bubble chart, radar chart) from a markdown string or a JSON config via fig(). Auto-layout, zero coordinates needed. Works in browser and Node.js.
---


# ai-figure Skill

Generates self-contained SVG diagrams. No coordinates needed — layout is computed automatically.

## Install

```bash
npm install ai-figure
```

## CDN (browser, no bundler)

```html
<script src="https://cdn.jsdelivr.net/npm/ai-figure/dist/index.global.js"></script>
<!-- or: https://unpkg.com/ai-figure/dist/index.global.js -->
<script>
  // Global: AiFigure.fig(...)
  document.getElementById('chart').innerHTML = AiFigure.fig('figure flow\na[A] --> b[B]');
</script>
```

## Usage

```typescript
import { fig } from 'ai-figure';

// Markdown string (preferred — compact, streaming-safe)
const svg = fig(`
  figure flow
  direction: LR
  palette: antv
  title: CI Pipeline
  subtitle: automated build and deploy
  code[Write Code] --> test{Tests Pass?}
  test --> build[Build Image]: yes
  test --> fix((Fix Issues)): no
  fix --> code
  build --> deploy[/Deploy/]
  group Pipeline: code, test, build
`);

// JSON config object (programmatic / strongly-typed)
const svg2 = fig({ figure: 'flow', nodes: [...], edges: [...] });

// DOM: document.getElementById('chart').innerHTML = svg;
// Node.js: fs.writeFileSync('chart.svg', svg);
```

`fig()` accepts either a **markdown string** or a **JSON config**. When given a string it never throws — partial or empty input (e.g. during AI streaming) returns a valid empty SVG that fills in progressively.

## Markdown syntax

**First line must be:** `figure <type>` — this is the required header, **not** a `key: value` config line.

Valid types: `flow` `tree` `mindmap` `arch` `sequence` `quadrant` `gantt` `state` `er` `timeline` `swimlane` `bubble` `radar`

Config lines use `key: value` syntax. Data lines use diagram-specific patterns.

| Key | Values | Default | Applies to |
|-----|--------|---------|------------|
| `title` | any string | — | all types |
| `subtitle` | any string | — | all types |
| `theme` | `light` `dark` | `light` | all types |
| `palette` | `default` `antv` `drawio` `figma` `vega` `mono-blue` `mono-green` `mono-purple` `mono-orange` | `default` | all types |
| `direction` | `TB` `LR` | `TB` | flow, tree, arch only |

Lines starting with `%%` are comments.

### Node notation (flow / tree / mindmap / arch)

| Notation | Shape |
|----------|-------|
| `id[label]` | process (rectangle) |
| `id{label}` | decision (diamond) |
| `id((label))` | terminal (pill) |
| `id[/label/]` | io (parallelogram) |
| `id` | process, id used as label |

### flow

```
figure flow
direction: LR
palette: antv
title: My Flow
subtitle: data pipeline example
A[Source] --> B[Target]          %% simple edge
A --> B[Target]: label           %% labeled edge
group Name: id1, id2, id3        %% logical group (dashed border)
```

### tree

```
figure tree
direction: LR
title: Org Chart
subtitle: company structure
root[Root]
root --> child[Child]
child --> leaf[Leaf]
```

### mindmap

```
figure mindmap
title: Product Strategy
subtitle: 2026 planning map
root[Product Strategy]
root --> market[Market]
root --> tech[Technology]
market --> smb[SMB]
market --> ent[Enterprise]
tech --> ai[AI Features]
```

### arch

```
figure arch
direction: TB
palette: antv
title: Web Stack
subtitle: three-tier architecture
layer Frontend
  ui[React App]
  assets[Static Assets]
layer Backend
  api[REST API]
  auth[Auth Service]
layer Data
  db[PostgreSQL]
```

### sequence

```
figure sequence
title: Login
subtitle: OAuth2 password flow
actors: Browser, API, DB         %% optional; inferred from messages if omitted
Browser -> API: POST /login      %% solid arrow
API --> Browser: 200 OK          %% dashed return arrow
```

### quadrant

```
figure quadrant
title: Priority
subtitle: effort vs value
x-axis Effort: Low .. High
y-axis Value: Low .. High
quadrant-1: Quick Wins    %% top-left
quadrant-2: Strategic     %% top-right
quadrant-3: Low Prio      %% bottom-left
quadrant-4: Long Shots    %% bottom-right
Feature A: 0.2, 0.9       %% label: x, y  (x/y in [0,1])
```

### gantt

```
figure gantt
title: Q1 Roadmap
subtitle: Jan – Mar 2025
section Design
  Wireframes: t1, 2025-01-06, 2025-01-24    %% label: id, start, end
  Mockups: t2, 2025-01-25, 2025-02-07
section Dev
  Frontend: t3, 2025-02-03, 2025-02-28
milestone: Launch, 2025-03-01
```

- Task format: `<label>: <id>, <yyyy-mm-dd>, <yyyy-mm-dd>` — **id is required**, even if you don't reference it
- `end` ≥ `start`; `section` groups tasks under a bold header; `milestone: <label>, <date>` marks a point in time

### state

```
figure state
title: Order Status
subtitle: e-commerce order lifecycle
idle[Idle]
processing[Processing]
accent: failed                   %% mark as accent/focal state
start --> idle                   %% start pseudo-state
idle --> processing: order placed
processing --> end: shipped
processing --> failed: error
failed --> idle: retry
```

- `id[label]` — normal state (rounded rectangle)
- `start` / `end` — reserved pseudo-state ids (filled circle / ringed circle)
- `id --> id2: event` — transition with optional label
- `accent: id` — mark a state as the focal/error state (max 1–2)

### er

```
figure er

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hustcc/ai-figure](https://github.com/hustcc/ai-figure) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
