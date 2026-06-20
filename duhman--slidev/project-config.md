---
trigger: always_on
description: |
---


# Slidev Presentations - Developer Slide Decks

Generate production-quality presentations using Slidev (sli.dev) v52+. Slidev renders Markdown + Vue as web-based slides with code highlighting, animations, diagrams, and themes. Output is a `slides.md` file that runs via `bun run dev`.

## When to Use

- User asks to create a presentation, slide deck, talk, or slides
- User wants to convert content (blog post, README, docs) into slides
- User needs a conference talk, lightning talk, or workshop deck
- User asks to modify or improve existing Slidev presentations

## Generation Workflow

Follow this checklist for every presentation:

1. **Clarify scope**: Ask about topic, audience, duration, and style if not specified
2. **Choose template**: Tech talk (20-30 min), lightning talk (5-10 min), or workshop (60+ min)
3. **Outline first**: Draft slide titles before writing content (share with user for approval)
4. **Write slides.md**: Generate the complete file with headmatter, layouts, and content
5. **Apply polish**: Add animations (v-click), code highlighting, transitions, speaker notes
6. **Setup project**: Create package.json if new project, or write to existing slides.md
7. **Verify**: Ensure `---` separators have blank lines above and below, frontmatter is valid YAML

### Content Guidelines

- **One idea per slide** - never overload
- **6x6 rule**: Max 6 bullet points, max 6 words each (for bullet slides)
- **Code blocks**: Keep under 15 lines per slide, highlight key lines
- **Progressive disclosure**: Use v-click to reveal points step-by-step
- **Speaker notes**: Add for every non-obvious slide
- **Visual variety**: Alternate between text, code, diagram, and image slides

## Project Setup

### New Project

```bash
bun create slidev@latest my-presentation
cd my-presentation
bun install
bun run dev
```

### package.json

```json
{
  "name": "my-presentation",
  "private": true,
  "packageManager": "bun@1.2.22",
  "scripts": {
    "dev": "slidev --open",
    "build": "slidev build",
    "export": "slidev export"
  },
  "dependencies": {
    "@slidev/cli": "^52.0.0",
    "@slidev/theme-default": "latest"
  }
}
```

### Directory Structure

```
my-presentation/
├── slides.md            # Main presentation (you generate this)
├── package.json
├── components/          # Custom Vue components (optional)
├── public/              # Static assets (images, videos)
├── global-bottom.vue    # Persistent footer (optional)
├── setup/               # Shiki, Monaco, Mermaid config (optional)
└── snippets/            # External code files for import (optional)
```

## Slide Structure

### Headmatter (First Block)

Controls the entire presentation. Always include:

```yaml
---
theme: default
title: "Presentation Title"
info: |
  ## Presentation Title
  A brief description for the info dialog.
author: "Author Name"
transition: slide-left
mdc: true
lineNumbers: false
fonts:
  sans: Inter
  mono: Fira Code
---
```

### Slide Separator

Every slide boundary requires `---` with **blank lines** above and below:

```markdown
# Slide 1 Content

---

# Slide 2 Content
```

### Per-Slide Frontmatter

```yaml
---
layout: center
transition: fade
class: text-center
background: /image.jpg
---
```

### Speaker Notes

```markdown
# My Slide

Content here

<!--
Speaker notes go in HTML comments at the end.
They support **Markdown** formatting.
Visible only in presenter mode (press `p`).
-->
```

### Importing Slides from Other Files

Use `src:` in frontmatter to import slides from external files:

```yaml
---
src: ./pages/intro.md
---
```

Import specific slides: `src: ./pages/demo.md#2,5-7`

Each imported file can contain multiple slides with their own frontmatter. Useful for splitting long presentations into manageable files.

## Layout Reference

All 19 built-in layouts:

| Layout            | Purpose                            | Key Props / Slots                            |
| ----------------- | ---------------------------------- | -------------------------------------------- |
| `default`         | General content                    | -                                            |
| `center`          | Centered content                   | -                                            |
| `cover`           | Title/cover page                   | `background`                                 |
| `intro`           | Introduction with author           | -                                            |
| `section`         | Section divider                    | -                                            |
| `end`             | Thank you / final slide            | -                                            |
| `fact`            | Highlight a statistic or fact      | -                                            |
| `statement`       | Bold statement/affirmation         | -                                            |
| `quote`           | Quotation display                  | -                                            |
| `full`            | Full-bleed (uses all screen space) | -                                            |
| `none`            | Blank canvas (no styling)          | -                                            |
| `image`           | Full-screen image                  | `image`, `backgroundSize` (default: `cover`) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duhman/slidev](https://github.com/duhman/slidev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
