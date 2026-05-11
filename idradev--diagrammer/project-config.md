---
trigger: always_on
description: Author Diagrammer JSON for the Diagrammer viewer. Use whenever the user asks for a mind map, concept map, flowchart, tree, dependency graph, ER diagram, or timeline. Output a single JSON document conforming to the schema below.
---


# Diagrammer authoring guide

You are producing a single JSON document that the Diagrammer viewer will
render on a pannable, zoomable canvas. Do not output prose around the JSON
unless the user explicitly asks for it; emit one fenced JSON block.

## Output contract

- Top-level keys: `version`, `type`, `title`, `nodes`, `edges`. Optional:
  `description`, `meta`.
- `version` MUST be the string `"1"`.
- `type` MUST be one of: `mindmap`, `tree`, `flowchart`, `graph`,
  `concept`, `timeline`. Pick the type that matches the user's intent — see
  the guidance below.
- `nodes[*].id` must be unique, stable, and URL-safe (`^[a-zA-Z0-9_-]+$`).
- `edges[*].from` and `edges[*].to` must reference existing node ids.
- You may include `x` / `y` (in pixels, node-center coordinates) when the
  user wants a hand-tuned layout. Otherwise omit them and let the viewer
  compute positions per type. Manual edits in the viewer's edit mode write
  these coordinates back when the user moves nodes.

## Choosing a type

| Type        | When to use                                                        |
| ----------- | ------------------------------------------------------------------ |
| `mindmap`   | Brainstorming, branches radiating from a single central topic.    |
| `tree`      | Strict hierarchy with one parent per child (org chart, taxonomy). |
| `flowchart` | Process or decision flow with directed steps.                     |
| `graph`     | Arbitrary network: dependencies, relationships, citations, **ER diagrams**. |
| `concept`   | Concept map with labelled, often bidirectional, links.            |
| `timeline`  | Linear sequence of events or milestones.                          |

If the user is ambiguous, ask one clarifying question — or default to
`mindmap` for "brainstorm" / "outline" intents, and `flowchart` for "process"
/ "how does X work" intents.

## Node fields

```ts
{
  id: string                  // unique, stable, URL-safe
  label: string               // short, ideally <= 40 chars
  description?: string        // single-line subtitle, <= 80 chars
  shape?:    'rectangle' | 'rounded' | 'ellipse' | 'diamond' | 'hexagon' | 'pill'
  color?:    'default' | 'slate' | 'blue' | 'green' | 'amber' | 'rose' | 'violet' | 'cyan'
  emphasis?: 'normal' | 'strong' | 'subtle'
  parent?:   string           // hint for tree/mindmap hierarchy
}
```

### Markdown formatting

Labels, descriptions, edge labels, and the map title render a small subset
of inline markdown. Use it to add typographic emphasis or to mark up
identifiers — not as decoration.

| Syntax          | Renders as                              |
| --------------- | --------------------------------------- |
| `**bold**`      | bold text                               |
| `*italic*`      | italic                                  |
| `_italic_`      | italic                                  |
| `` `code` ``    | inline monospace, e.g. for identifiers  |
| `~~strike~~`    | strikethrough                           |
| `[text](url)`   | link (opens in a new tab)               |

Nesting is allowed (`**` `` `code` `` `**`). Markup that doesn't match these
delimiters renders literally — no headings, lists, paragraphs, or HTML.
Avoid links inside node labels; put them in `description` instead.

### Visual conventions (use sparingly)

- `emphasis: "strong"` on the central node of a mindmap or the root of a tree.
- `shape: "diamond"` for decisions/branches in flowcharts.
- `shape: "pill"` for terminal/IO nodes (start, end, external service).
- `color: "rose"` for failure / error / risk nodes; `color: "green"` for
  success; `color: "amber"` for warnings or caveats. Use color to convey
  meaning, not decoration.
- Leave most nodes at `shape: "rounded"` and `color: "default"`. Restraint
  reads as design.

## Edge fields

```ts
{
  from: string                // node id
  to:   string                // node id
  label?:     string          // edge label, e.g. "yes", "no", "depends on"
  style?:     'solid' | 'dashed' | 'dotted'
  direction?: 'forward' | 'backward' | 'both' | 'none'
}
```

- Default direction is `forward` (single arrow at `to`).
- Use `dashed` for weak / conditional / async links; `dotted` for very weak
  or implied relationships.

## Sizing guidance

- 5–25 nodes per map is the sweet spot. Past 30, the viewer remains usable but
  comprehension drops. Split into multiple maps if the user's request implies
  more.
- Keep labels concise. Move detail into `description`. Move long explanations
  out of the map entirely.
- Avoid cycles in `tree` and `flowchart`. Use `graph` or `concept` if
  cycles are essential.

## Multiple groups in one map

A single map can contain **multiple disconnected sub-diagrams** that share the
canvas without being linked. The viewer detects connected components
automatically (via edges + `parent` hints) and lays each out independently,
packing them side-by-side with padding. No special syntax is required:

- Just emit the nodes and edges for each group. Anything that has no edge or
  `parent` link to another group becomes its own cluster.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IdraDev/Diagrammer](https://github.com/IdraDev/Diagrammer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
