---
trigger: always_on
description: > **You are reading this because you need to render a UI wireframe mockup.**
---

# Wireloom for AI Agents

> **You are reading this because you need to render a UI wireframe mockup.**
> Wireloom is a small text DSL for sketching user-interface layouts as inline SVG. This file is written for LLM agents (Claude, Codex, Cursor, etc.) that need to author wireframes — copy or link it into your agent's context when working on UI design tasks.

## What is Wireloom?

Wireloom is a small indentation-based text language for UI wireframe mockups. You write a layout as indented plain text inside a ```wireloom fenced code block, and the Wireloom renderer turns it into an SVG wireframe. Output is monochrome, sketch-style — it reads as a mockup, not a finished UI.

Because Wireloom outputs plain SVG, the same block renders in **GitHub, Obsidian, Notion, static site generators, or any Markdown tool that handles SVG**. The read side needs no per-host plugin — only the authoring side needs the renderer (via `npm install wireloom` or a tool that bundles it).

## When to Use Wireloom

Use Wireloom when the user asks for:

- "Mock up a screen / dialog / settings page / sign-in form…"
- "Draw a wireframe for…"
- "Sketch the layout of the new UI"
- "Show me how this feature would look"
- "Diagram the toolbar / inspector / split view"
- Any request about the **shape of a user interface** that isn't already a running app

## When NOT to Use Wireloom

| If the user wants… | Use instead |
|--------------------|-------------|
| Flowchart, sequence, class, ER, or state diagram | `mermaid` fenced block |
| Interactive prototype you can click through | A real component in their frontend framework (React, Svelte, Vue…) |
| Architecture / concept / dependency map with freeform nodes | A whiteboard/canvas tool or Mermaid graph |
| A real, working form or tool | Actual code in the target framework |

Wireloom is strictly for **static structural mockups**. If the ask is "can you make this tool / dashboard / page actually work?", write the real component instead.

## Minimum Viable Wireframe

```
window:
  text "Hello, Wireloom"
```

Rendered: a bordered window containing the text.

## Primitives (v0.4 / v0.4.1 / v0.4.5 / v0.50)

Every source must start with a single `window` root. In v0.4.1, one or more `annotation` nodes may follow the `window` as siblings to add user-manual-style callouts (see [Annotations (Callouts)](#annotations-callouts--v041)). v0.4.5 adds form controls (checkbox/radio/toggle), file trees, menubars, breadcrumbs, chips, avatars, and inline status indicators — see [v0.4.5 Primitives](#v045-primitives). v0.50 adds the mobile-navigation primitive set (`spacer`, `navbar`, `tabbar`, `tabitem`, `backbutton`, `sheet`, `segmented`, `segment`) plus `row justify=`, `header large`, and the `chevron` flag on `slot`/`item`: see [v0.50 Primitives](#v050-primitives).

### Structural containers

| Primitive   | Children?                   | Positional args            | Purpose |
|-------------|-----------------------------|----------------------------|---------|
| `window`    | Yes                         | optional title string      | Root container. Exactly one per source. |
| `header`    | Yes                         | —                          | Top chrome band (title-level content). |
| `footer`    | Yes                         | —                          | Bottom chrome band (actions), or optional last child of a `slot`. |
| `panel`     | Yes                         | —                          | Bordered dashed content container. |
| `section`   | Yes                         | required title string      | Labeled container with small-caps title band. Supports `badge="…"`, `accent=`. |
| `tabs`      | Yes (`tab` children only)   | —                          | Tab-bar container. |
| `row`       | Yes                         | —                          | Horizontal flow. Supports `align=left|center|right`. |
| `col`       | Yes                         | optional pixel width or `fill` | Vertical flow. Default is `fill` when no width given. |
| `list`      | Yes (`item`/`slot` only)    | —                          | Vertical list container. |
| `slot`      | Yes                         | required title string      | Titled bordered card. Supports `active` flag, `state=`, `accent=`, optional trailing `footer:` child. |
| `grid`      | Yes (`cell` children only)  | —                          | Fixed `cols=N rows=M` grid. Cells auto-flow or take explicit `row=`/`col=`. **v0.4** |
| `resourcebar` | Yes (`resource` children only) | —                      | Horizontal resource strip for game-UI headers. **v0.4** |
| `stats`     | Yes (`stat` children only)  | —                          | Terse inline stat strip (LABEL value). **v0.4** |
| `navbar`    | Yes (`leading:`/`trailing:` only) | —                    | Top chrome band with `leading:` and `trailing:` slots on one line. Direct child of `window` only, mutually exclusive with `header`. **v0.50** |
| `tabbar`    | Yes (`tabitem` children only) | —                        | Bottom chrome band for primary mobile navigation. Direct child of `window` only, mutually exclusive with `footer`. **v0.50** |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [StardockCorp/Wireloom](https://github.com/StardockCorp/Wireloom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
