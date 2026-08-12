---
trigger: always_on
description: Spatial Deck is a single-file HTML presentation framework. Everything lives in `index.html` — CSS, JS, config, content. No build process, no npm, no bundler.
---

# Spatial Deck — Codex Instructions

## What This Is

Spatial Deck is a single-file HTML presentation framework. Everything lives in `index.html` — CSS, JS, config, content. No build process, no npm, no bundler.

## Before You Start

**Read `HANDOFF_PROMPT.md`** in the repo root. It has the full architecture guide, conventions, and common tasks. This file gives you behavioral rules; the handoff prompt gives you technical context.

## Rules

### File Structure
- **Never split `index.html` into multiple files.** The single-file design is intentional.
- **Never add npm/webpack/vite/etc.** Zero build process is a core feature.
- The only external dependency is Three.js via CDN import map (optional, for constellation map).

### Editing SECTIONS
- The `SECTIONS` array near the top of the second `<script>` block drives all slide content.
- Changing SECTIONS automatically regenerates slides on page load.
- Use `\n` for line breaks in titles. Use `<br>` and `<br><br>` in taglines.
- Use `\n` for line breaks in bullets.
- `img: 'MEDIA_CYCLER'` requires a matching IIFE after the build loop.
- `img: 'path/to/file.jpg'` renders as a standard `<img>` tag (no media cycler, no pixelated reveal). Use `MEDIA_CYCLER` with a single-item array if you want the canvas-based reveal.
- `img: 'IFRAME:url'` embeds a responsive iframe.
- `img: ''` shows a gradient placeholder.
- Build loop priority: `IFRAME:` prefix → iframe, real path → `<img>` tag, `MEDIA_CYCLER` → media cycler mount, empty → gradient placeholder.

### Processing Annotation Dumps

When the user pastes a `# Annotations` block (exported from the deck's annotation panel, `A` key), it's a batch of edits to bake into source. Each entry looks like:

```
## N. <context> #<slide-num>
**Selector:** `<css-selector>`
**left:X%, top:Y% on slide #N**
**Note:** TEXT EDIT: "<truncated old>…" → "<truncated new>…"
**Full text:** <authoritative new text>
```

**Annotation type → action:**

| Type | How to recognize | Action |
|------|------------------|--------|
| **Text edit** | `Note: TEXT EDIT: ...` + `**Full text:**` line | Replace in source with **Full text** *verbatim*. The truncated `Note:` line is preview only — ignore it for the actual replacement. |
| **Transform** | `**TRANSFORM left:X%, top:Y% · translate(...)**` line | **Usually no-op.** Already in user's localStorage; applies at runtime. Only bake into source if user explicitly asks ("make this permanent for everyone"). |
| **Bare `Note:`** | Plain note, no `TEXT EDIT:` prefix | Ambiguous. Most common pattern: the note IS the new text content for the selected element (e.g., on an SVG `<text>` node, `Note: scope "locked"` means *set the node's text to that string*). Cross-reference the builder code. If still unclear, **ask** — don't guess. |
| **Settings block** | `## Current Settings (slide 0)` block at top of dump | If non-empty, bake values into the `D` defaults object near slide 0. Empty = no change. |

**Source-of-truth varies per annotation:** SECTIONS array (most common), layout builders, the BONUS object, the speaker-notes function, hardcoded SVG `<text>` strings inside builders. Read the selector and the surrounding builder code before editing.

**Workflow:**
1. Group edits by source location.
2. Apply with Edit/Write. The PostToolUse hook (`.Codex/hooks/validate-js.sh`) parses all inline `<script>` blocks after each edit and blocks Codex if a token-level syntax error was introduced.
3. For layout-affecting changes (anything inside `placed`, `sg-collage`, or grid-based layouts), spawn a preview agent and screenshot before committing. Text-only edits in SECTIONS don't need preview.
4. One commit per annotation batch; message states the count and types.

**Gotchas (codified from prior incidents):**

1. **The Edit tool silently converts ASCII `'` to Unicode curly `'`/`'` in replacement text.** JavaScript doesn't accept curly quotes as string delimiters → syntax error. The hook catches it now, but to avoid the round-trip when editing JS strings: use a Python binary-replace, e.g. `data.replace('‘'.encode(), b"'").replace('’'.encode(), b"'")`. Preserve mid-string apostrophes that are valid U+2019 content (only fix the *delimiter* positions).
2. **Apostrophes inside JS single-quoted strings need `’`, not raw `'`.** A literal `'` mid-string terminates the string. Match the existing codebase style (`you’ve`, `we’re`).
3. **Ambiguous freeform notes get a question, not a guess.** "Let's get another image here" → ask which image and where.
4. **Index-based class assignments break under array inserts.** If a builder does `class="foo-${i%N}"` and those classes carry differential styles (grid spans, colors, animations), inserting in the middle of the array rotates all subsequent class assignments. Two safe patterns:
   - **Append at end** — only if visual order doesn't matter.
   - **Add a separate field** the builder renders with an explicit-position CSS class. (Pattern: `extraTile` config field → builder emits a tile with class `.sg-tile-fill-tr{grid-column:6;grid-row:1}` → slots without disturbing the auto-flow of other tiles.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ibrews/spatial-deck](https://github.com/ibrews/spatial-deck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
