---
trigger: always_on
description: This is a small static front-end demo for a PPT-like animated mind map.
---

# AGENTS.md

## Project Overview

This is a small static front-end demo for a PPT-like animated mind map.

- Project data lives in `project/source.js`, which exports the unordered-list Markdown tree consumed by `src/main.js`.
- The tree is traversed in preorder.
- Nodes are rendered as HTML elements so their boxes can grow to contain text.
- Links are rendered as SVG curves behind the HTML nodes.
- The UI is plain HTML/CSS/JS, with no build step or runtime dependencies.

## Markdown Data Rules

- Each tree node is one unordered-list Markdown item.
- A list item may use an indented continuation line for a two-line label:

```md
- Markdown Mindmap
  项目汇报思维导图演示
    - 需求分析
      用户目标与演示场景
```

- For two-line labels, the first line is the subtitle and the second line is the main title.
- The top-left deck heading uses the root node the same way:
  - root first line -> eyebrow/subtitle
  - root second line -> main title
- Node boxes also use the same two-line convention:
  - first line -> small subtitle
  - second line -> normal-size title
- Single-line labels render as a normal one-line node title.
- Control readouts may collapse multiline labels into an inline preview such as `副标题 / 主标题`.
- A node may optionally attach one illustration with an `@image` metadata continuation line. Use paths relative to `project/`:

```md
- 展示设计
  画布布局与动画策略
  @image layout.svg
```

- `@image` lines are metadata only:
  - They do not appear in node text.
  - A node may have at most one image.
  - Supported image formats are whatever browser `<img>` supports; use PNG, JPG/JPEG, or SVG for project assets.
  - Prefer short local paths such as `example.svg`, `example.png`, or `image-asset-1/a.jpg`; they resolve to `./project/...`.
  - For example, `@image image-asset-1/a.jpg` resolves to `./project/image-asset-1/a.jpg`.
  - Explicit relative paths such as `./project/image-asset-1/a.jpg`, absolute paths, data URLs, and HTTP(S) URLs remain supported when needed.
  - If multiple `@image` lines are added to one node, the latest parsed value wins.
- Illustrations render inside their node card:
  - selected image nodes show the image expanded below the node text
  - non-selected image nodes show a small thumbnail below the node text
  - nodes without `@image` do not reserve image space
- Image expansion follows the real selected preorder node only. Clicking a node to move the camera must not expand its image unless it also changes `activeIndex`.

## Running And Checking

- Start local dev server: `npm run dev`
- Syntax check: `npm run check`
- Dev URL: `http://127.0.0.1:5173/`

The dev server is a dependency-free Node.js static server. If `5173` is occupied by a stale process, restart that process before validating in the browser.

## Core Files

- `index.html`: page shell and top controls.
- `project/source.js`: project Markdown data. Replace this file to change the mind map content.
- `project/`: project Markdown data and local assets referenced by `@image`.
- `src/main.js`: imports project data, parses Markdown, handles preorder navigation, layout model, HTML node sync, SVG link sync.
- `src/styles.css`: page styling, node/link styling, slider styling, animations.
- `p.md`: original product prompt/spec.

## Interaction Rules

- Up/down arrow keys move to previous/next preorder node.
- Top arrow buttons do the same.
- The range slider jumps directly to a preorder index.
- The zoom slider controls camera distance, scaling the whole canvas from about `70%` to `140%`; default is `100%`.
- The second control row shows the current node label and next node label.
- Clicking a visible node moves the camera toward that node's current-layout position without changing the selected node or expanded image. Move the camera as little as possible: if the clicked node is already inside the central 40% of the viewport, do not move; otherwise shift just enough to bring it into that central band.
- Changing the selected node should use the same central 40% camera rule: move as little as possible to bring the selected node into that central band.

Keep all navigation paths going through `setActiveIndex()` so buttons, keyboard, slider, graph, and counter stay synchronized.

## Layout Rules

- The current path from `root` to selected node is horizontal.
- Already visited but non-path branches appear above their parent and preserve tree structure.
- Unvisited nodes are completely hidden and occupy no layout space.
- The horizontal selected path should stay visually stable.
- Default node and text sizing is intentionally large, roughly 30% larger than the original compact demo.
- The camera pans across a fixed logical canvas and must not auto-scale nodes or text because of browser aspect ratio. Node and font sizes stay in CSS pixels unless the user changes the zoom slider.
- The zoom slider is the only intended way to scale the whole canvas.
- The visible viewport uses the actual `#mindmap` element size. The presentation stage should stretch with the browser window.
- `layout.centerBaseline = 520` controls the horizontal path's baseline in logical canvas coordinates.
- Completed branches are allowed to exceed the viewport and be clipped. Do not scale the camera view to fit them, because that makes nodes and text smaller.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agegr/mindmap-ppt](https://github.com/agegr/mindmap-ppt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
