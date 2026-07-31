---
trigger: always_on
description: This file defines how to write blog posts for the Effection website
---

# Effection Blog — Writing Agent Guide

This file defines how to write blog posts for the Effection website
(`www/blog/`). It covers voice, structure, and technical accuracy constraints.

The Effection blog is distinct from the Frontside company blog. Posts here are
shorter, more focused, and always grounded in what Effection actually does.

All Effection blog posts inherit the **shared Frontside voice** defined in
[`frontside.com/AGENTS.md`](https://github.com/thefrontside/frontside.com/blob/main/AGENTS.md).
That guide covers sentence rhythm, verbal tics, humor/metaphor patterns, and
four voice profiles (Opinion, Tutorial, Consultative, Narrative). This file adds
Effection-specific constraints: technical accuracy rules, shorter post length
(500-800 words), and Taras as the default author voice.

## File Conventions

- **Directory pattern:** `www/blog/YYYY-MM-DD-slug/index.md`
- **Slug format:** lowercase, hyphen-separated, derived from title
- **Images:** placed in the same directory as the post's `index.md`
- **Frontmatter:**

```yaml
---
title: "The Title of the Post"
description: "A 1-2 sentence pitch that makes someone want to read the post."
author: "Author Name"
tags: ["tag1", "tag2"]
image: "featured-image.svg"
---
```

- The title is NOT repeated as an `# H1` inside the post body.
- Tags are lowercase strings, typically 1-3.
- Image paths are relative to the post directory.

## Featured Images

- **Template:** `www/blog/blog-image-template.svg`
- **Dimensions:** 1200×630 (standard OG image size)
- **Color scheme:** Automatically adapts to system light/dark mode via
  `@media (prefers-color-scheme: dark)` inside the SVG `<style>` block.
- **Usage:** Copy the template into your post directory, rename it, and
  customize the title, caption, diagram area, and legend items.
- **Colors:** Use the CSS classes defined in the template (`svg-card`,
  `svg-scope-parent`, `svg-scope-child`, `svg-pill`, `svg-mono`,
  `svg-dot-primary`, etc.) — do NOT hardcode `fill`/`stroke` values that would
  break in one mode.
- **Class prefix:** All CSS classes in SVG images MUST use the `svg-` prefix
  (e.g., `svg-grid`, `svg-title`, `svg-card`). This prevents namespace
  collisions when the SVG is inlined into the page DOM by the `inline-svg`
  plugin — without the prefix, generic class names like `.grid` or `.label` leak
  into the page and break other elements.
- **Diagram patterns:** The template includes three commented-out diagram
  examples: nested boxes (scope ownership), flow arrows (pipelines/sequences),
  and stacked items (feature lists). Uncomment the one closest to your needs and
  customize.
- **Shadows/filters:** Light and dark mode use separate shadow filters
  (`softShadow-light`/`softShadow-dark`, `tinyShadow-light`/`tinyShadow-dark`).
  Elements that need shadows must be duplicated in two `<g>` wrappers — one with
  class `svg-shadow-light` and one with `svg-shadow-dark`. CSS toggles
  visibility.
- **Background:** Uses show/hide pairs (`svg-bg-light`/`svg-bg-dark`,
  `svg-glow-light`/`svg-glow-dark`) since CSS cannot change SVG gradient stop
  colors.

## Voice

### Primary author: Taras Mankovski

The default voice for Effection blog posts is Taras'. Here's how it works.

**Register:** Clean, direct prose. Not academic, not marketing. Technically
precise but accessible. The authority comes from having built the thing, not
from credentials.

**Goal:** Make the reader understand that Effection makes async feel normal.
That's the core message. Everything else supports it.

## Audience

Primary audience for Effection blog posts is Hacker News and r/javascript:
technically fluent, skeptical, and quick to nitpick imprecision.

- Prefer concrete, falsifiable claims over attributed motives ("TC39 wanted…").
- Define overloaded terms once (e.g. "scope" meaning lifetime/ownership).
- Avoid bikeshed triggers when a tighter semantics framing will do.
- Keep examples consistent with the hook and with real runtime behavior.

**Point of view:**

- First person singular ("I") for opinion and advocacy posts.
- First person plural ("we") for project announcements and release notes.
- Second person ("you") when addressing the reader's situation.
- Mix all three when a post is both advocating and explaining.

**Prose style:**

- Clean prose mostly. Every sentence earns its place.
- Metaphors and color are welcome when they serve the argument — don't force
  them, but don't strip them out either. If a reviewer adds a good metaphor
  during editing, keep it.
- No filler. No "In today's rapidly evolving landscape" openings. No padding.
- Short posts (500-800 words). Get in, make the point, get out.

**Opening style:**

- Prefer grounding technical arguments in history or foundational context first,
  then connecting to today. The goal is to show that the idea is fundamental,
  not novel.
- Don't lead with "Effection is..." — lead with the problem or the principle.

**Code:**

- Light: 1-3 well-chosen snippets per post.
- Code reinforces the argument. It's evidence, not tutorial steps.
- Show the Effection way. Don't show verbose "before" code unless the contrast
  is the point.

**Citations and links:**

- Link non-obvious claims inline. Common knowledge stays unlinked.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thefrontside/effection](https://github.com/thefrontside/effection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
