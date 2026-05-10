---
trigger: always_on
description: Slidev is a web-based slides maker and presenter designed for developers. It allows you to create presentations by writing content in Markdown. The main content file is `[slides.md](mdc:slides.md)`.
---

# Creating Slides with Slidev

Slidev is a web-based slides maker and presenter designed for developers. It allows you to create presentations by writing content in Markdown. The main content file is `[slides.md](mdc:slides.md)`.

## Slide Separators

Use `---` padded with a new line to separate your slides.

```markdown
# Slide 1 Title

Content for slide 1.

---

# Slide 2 Title

Content for slide 2.
```

## Frontmatter

Each slide (and the presentation itself) can be configured using YAML frontmatter, placed either at the very beginning of `[slides.md](mdc:slides.md)` for global settings or between slide separators for slide-specific settings.

Common frontmatter options:

*   `title`: Title of the presentation or a specific slide.
*   `info`: Additional information about the presentation (Markdown enabled).
*   `theme`: Specifies the theme (e.g., `seriph`, `default`).
*   `background`: Sets the background image or color for a slide.
    *   Example: `background: https://source.unsplash.com/collection/94734566/1920x1080`
*   `class`: Applies UnoCSS utility classes to the slide.
    *   Example: `class: text-center`
*   `layout`: Defines the layout for the slide (e.g., `two-cols`, `image-right`, `center`).
    *   Example: `layout: two-cols`
*   `layoutClass`: Applies classes to the layout container.
    *   Example: `layoutClass: gap-16` (used with `two-cols`)
*   `transition`: Specifies the slide transition effect (e.g., `slide-left`, `fade-out`, `slide-up`).
    *   Example: `transition: slide-left`
*   `level`: Sets the heading level for the slide title in a Table of Contents.
    *   Example: `level: 2`
*   `drawings`: Configures drawing features.
    *   Example: `drawings: persist: false`
*   `mdc`: Enables MDC (Markdown Components) syntax, a powerful way to use Vue components and apply styles/classes directly in Markdown. See dedicated section below.
    *   Example: `mdc: true`
*   `src`: Imports content from another Markdown file.
    *   Example: `src: ./pages/another-part.md`
*   `hide`: Hides a slide from the presentation.
    *   Example: `hide: true`
*   Other custom frontmatter can be defined by themes or for specific needs (e.g., `image` for image layouts, `dragPos` for draggable elements).

Example of global frontmatter:
```yaml
---
title: My Awesome Presentation
theme: seriph
info: |
  More details about this presentation.
---

# First Slide
...
```

Example of slide-specific frontmatter:
```markdown
---
layout: image-right
image: https://example.com/image.jpg
transition: fade-out
---

# Slide with Image Right

This slide has a specific layout and background.
```

## MDC (Markdown Components) Syntax

When `mdc: true` is enabled in your frontmatter (globally or per slide), you can use MDC syntax. This allows for a more concise and powerful way to embed Vue components and apply attributes or classes directly to Markdown elements or components.

It's often considered the easiest way to apply styles and classes.

Example:
```markdown
<!-- Standard Markdown link -->
[Slidev](mdc:https:/sli.dev)

<!-- MDC equivalent, allowing classes or other attributes -->
#title[My Title with MDC]
Regular text.
::div{.p-4 .bg-red-100}
This is a div with padding and a red background, using MDC.
::

<!-- Using a component with MDC -->
::Counter{count=5 initial=2}
::
```
Consult the official [MDC Syntax documentation](mdc:https:/sli.dev/features/mdc) for more details on its capabilities.

## Markdown and HTML

Standard Markdown syntax is fully supported. You can also use HTML directly within your slides for more complex structures or styling.

```html
<div class="mt-4 p-2 bg-gray-100 rounded">
  This is an HTML div with some classes.
</div>
```

## Vue Components

Embed Vue components directly in your Markdown for interactivity. Slidev provides built-in components (e.g., `<Tweet id="..."/>`, `<Youtube id="..."/>`) and you can create your own in the `components/` directory.

```html
<!-- Using a built-in component -->
<Tweet id="1390115482657726468" />

<!-- Using a custom component (e.g., ./components/Counter.vue) -->
<Counter :count="10" />
```

## Code Blocks

Display code snippets with syntax highlighting.

```typescript
function greet(name: string) {
  console.log(`Hello, ${name}!`);
}
```

### Line Highlighting & TwoSlash

Specify line highlights, diffs, or enable TypeScript hover information with TwoSlash.

```typescript {all|1|2-3} twoslash
// TwoSlash enables TypeScript hover information
import { ref } from 'vue'
const count = ref(0) // This line will be highlighted
```

### Shiki Magic Move

Animate changes between multiple code snippets by wrapping them with ` `````md magic-move ```` ` (four backticks).

````markdown
````md magic-move
```ts
// Step 1
let a = 1;
```

```ts
// Step 2
let a = 1;
let b = 2; // Added this line
```
````
````

### Monaco Editor

Embed a fully functional Monaco editor.

*   `{monaco}`: Turns a code block into an editor.
*   `{monaco-run}`: Creates an editor that can execute the code.

```typescript {monaco-run}
console.log("Hello from Monaco Editor in Slidev!");
```

## Animations

### Click Animations


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sanjeed5/cursor-slidev](https://github.com/sanjeed5/cursor-slidev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
