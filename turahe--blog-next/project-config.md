---
trigger: always_on
description: This repository contains a personal developer blog and portfolio built with:
---

# AGENTS.md

# Personal Blog & Portfolio — AI Coding Agent Instructions

## 1. Project Overview

This repository contains a personal developer blog and portfolio built with:

* Next.js
* TypeScript
* Tailwind CSS
* MDX/content-based blog
* Scroll World cinematic experience
* Responsive UI
* SEO-focused architecture
* Performance-focused frontend

The project is a **content-first engineering portfolio**, not an animation showcase.

The website must remain useful, accessible, indexable, and performant even when the Scroll World experience is unavailable.

---

# 2. Source of Truth

Before making architectural or implementation decisions, read the relevant documentation.

Required documents:

```text
PRD.md
DESIGN.md
```

Relevant skills:

```text
.cursor/skills/ui-design/SKILL.md
.cursor/skills/scroll-world/SKILL.md
.cursor/skills/seo/SKILL.md
.cursor/skills/performance/SKILL.md
```

Additional documentation may exist under:

```text
docs/
```

Do not invent requirements that contradict these documents.

When requirements conflict, use this priority:

```text
PRD.md
    ↓
DESIGN.md
    ↓
Architecture decisions
    ↓
Skills
    ↓
Implementation details
```

---

# 3. General Agent Behavior

Before changing code:

1. Inspect the repository.
2. Understand the existing architecture.
3. Read relevant documentation.
4. Identify affected components.
5. Determine whether the change affects:

   * UI
   * SEO
   * performance
   * Scroll World
   * accessibility
   * content
6. Make the smallest reasonable change.
7. Run relevant validation.
8. Review the resulting implementation.

Do not rewrite unrelated code.

Do not introduce dependencies without justification.

Do not create abstractions prematurely.

---

# 4. Required Skill Selection

Use the following rules.

## UI changes

Read:

```text
.cursor/skills/ui-design/SKILL.md
```

Examples:

* components
* layouts
* navigation
* buttons
* cards
* typography
* responsive design
* dark mode
* animations

---

## Scroll World changes

Read:

```text
.cursor/skills/scroll-world/SKILL.md
.cursor/skills/ui-design/SKILL.md
.cursor/skills/performance/SKILL.md
```

Examples:

* scene
* camera
* video
* scroll animation
* cinematic hero
* visual transitions
* scene assets

Scroll World changes MUST also consider performance.

---

## SEO changes

Read:

```text
.cursor/skills/seo/SKILL.md
```

Examples:

* metadata
* sitemap
* robots
* canonical
* structured data
* Open Graph
* article metadata
* internal linking

---

## Performance changes

Read:

```text
.cursor/skills/performance/SKILL.md
```

Examples:

* bundle size
* Core Web Vitals
* image optimization
* video optimization
* lazy loading
* dynamic imports
* hydration
* client/server boundaries
* scroll performance

---

## Blog/content changes

Read:

```text
.cursor/skills/seo/SKILL.md
.cursor/skills/ui-design/SKILL.md
```

Examples:

* MDX
* articles
* categories
* tags
* article pages
* article layout
* table of contents

---

# 5. Cross-Skill Rules

Some changes require multiple skills.

## Homepage

Read:

```text
ui-design
scroll-world
performance
seo
```

---

## Blog Article

Read:

```text
ui-design
seo
performance
```

---

## Project Case Study

Read:

```text
ui-design
seo
performance
```

---

## Scroll World Hero

Read:

```text
ui-design
scroll-world
performance
seo
```

---

# 6. Architecture Principles

## Server Components First

Next.js Server Components are the default.

Do not add:

```tsx
"use client"
```

unless client-side behavior is required.

Client Components are appropriate for:

* browser APIs
* interactive UI
* animation
* Scroll World
* client state
* event handlers

Keep Client Component boundaries as small as practical.

---

# 7. Content Architecture

Content must remain independent from presentation.

Preferred:

```text
content
   ↓
data/model
   ↓
page
   ↓
components
```

Do not embed large amounts of blog/project content directly inside reusable UI components.

---

# 8. Blog Architecture

Blog content should preferably be stored as MDX or another static content format.

Example:

```text
content/
└── blog/
    ├── article-one/
    │   └── index.mdx
    ├── article-two/
    │   └── index.mdx
    └── article-three/
        └── index.mdx
```

Each article should have frontmatter.

Example:

```yaml
---
title: "Designing Reliable Go Services"
description: "..."
date: "2026-08-01"
tags:
  - Go
  - Architecture
category: "Software Engineering"
draft: false
---
```

Do not expose draft articles publicly.

---

# 9. Project Architecture

Projects should support case-study content.

Recommended structure:

```text
content/
└── projects/
    ├── project-one/
    ├── project-two/
    └── project-three/
```

A project should be able to contain:

* overview
* problem
* solution
* architecture
* implementation
* technologies
* challenges
* results
* lessons learned

---

# 10. UI Architecture

Prefer:

```text
components/
├── layout/
├── ui/
├── hero/
├── blog/
├── projects/
└── scroll-world/
```

Do not create one giant:

```text
components/HomePage.tsx
```

containing the entire website.

---

# 11. Design System

`DESIGN.md` is the visual source of truth.

Use centralized design tokens.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [turahe/blog-next](https://github.com/turahe/blog-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
