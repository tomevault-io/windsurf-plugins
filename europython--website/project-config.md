---
trigger: always_on
description: > **Ask if unsure. Suggest improvements. Never introduce random colors or
---

# EuroPython 2026 Website — Agent Context

> **Ask if unsure. Suggest improvements. Never introduce random colors or
> fonts.**

The official **EuroPython 2026** conference website (`ep2026.europython.eu`),
built with **Astro 6** + **pnpm** + **Tailwind CSS v4** + **Svelte 5**.

**Worktree**: `/home/user/workspace/EuroPython/ep2026-ai` — branch `ep2026-ai`,
forked from the main repo at `/home/user/workspace/EuroPython/website/`.

---

## Quick Start

```bash
make install              # install deps (pnpm 9.3)
make dev                  # dev server → http://localhost:4321
make build                # full build + type-check + pagefind
pnpm format               # Prettier (prettier-plugin-astro)
pnpm astro check          # TypeScript validation only
```

---

## 🚨 DO / DON'T

### DO ✅

**Content & pages**

- **DO** create new pages as `.md` files — MDX is deprecated, migrate existing
  `.mdx` when touched
- **DO** create pages as either `src/content/pages/page-name.md` or
  `src/content/pages/page-name/index.md`
- **DO** prefix hidden/internal pages with underscore: `_draft.md`,
  `_private-section/`

**Architecture**

- **DO** split Astro pages into **page → sections → components** (see "Page
  Architecture" below)
- **DO** keep pages thin — they should only set layout, fetch data, and compose
  sections
- **DO** create page-block components in `src/components/sections/`
- **DO** create reusable UI primitives in `src/components/ui/`

**Styling**

- **DO** use Tailwind utility classes from the existing `@theme` tokens in
  `src/styles/tailwind.css`
- **DO** use `<style>` blocks in `.astro` components for scoped component CSS
- **DO** import `global.css` for base resets and reduced-motion

**Process**

- **DO** add **nikoshell** as reviewer when modifying files outside
  `src/content/`
- **DO** run `make dev` or ` build` before claiming something works
- **DO** ask questions when requirements are ambiguous or you're unsure which
  token/component to use
- **DO** suggest improvements when you spot gaps, duplication, dead code, or
  inconsistencies

### DON'T ❌

- **DON'T** introduce new fonts — use only `--font-heading`, `--font-body`,
  `--font-sans` from `tailwind.css`
- **DON'T** introduce new colors — use only `--color-*` tokens from the `@theme`
  block in `tailwind.css`
- **DON'T** inline hex, rgb, or oklch values in components
- **DON'T** create new `.mdx` files
- **DON'T** put everything in one monolithic page file — a page orchestrates
  sections
- **DON'T** create a new section component if it can be composed from existing
  `ui/` primitives
- **DON'T** add `<style>` blocks without checking if a global or utility class
  already covers it
- **DON'T** create new layout files unless `Layout.astro`,
  `MarkdownLayout.astro`, `ScheduleLayout.astro`, or `SectionLayout.astro` don't
  fit
- **DON'T** remove or rename undocumented files without checking all import
  references

---

## Tech Stack

| Layer           | Technology                                                      |
| --------------- | --------------------------------------------------------------- |
| Framework       | **Astro 6** (static site generation)                            |
| Components      | **Svelte 5** for interactive islands                            |
| Styling         | **Tailwind CSS v4** (CSS-first config, no `tailwind.config.js`) |
| Package Manager | **pnpm 9.3**                                                    |
| Icons           | Font Awesome Free 6 (`@fortawesome/fontawesome-free`)           |
| State           | Nano Stores (`nanostores`)                                      |
| Search          | Pagefind (via `astro-pagefind`)                                 |
| Language        | TypeScript 5.x (strict)                                         |
| Formatting      | Prettier + `prettier-plugin-astro`                              |

---

## Design System: Colors & Fonts

**Single source of truth**: `src/styles/tailwind.css` → `@theme` block.
Everything is in oklch with hex/rgba in comments for reference.

### How to use in markup

```astro
<!-- ✅ Correct: existing tokens via Tailwind classes -->
<div class="bg-surface-subtle text-accent border-border">
  <h2 class="text-accent font-heading">Title</h2>
</div>

<!-- ❌ Wrong: ad-hoc values -->
<div style="background: #123; color: rgb(200,200,200);"></div>
```

### Key surface & text tokens

| Tailwind class         | Purpose                                           |
| ---------------------- | ------------------------------------------------- |
| `bg-bg`                | Base page background (very dark navy)             |
| `bg-section-bg`        | Section background                                |
| `bg-card`              | Elevated card background                          |
| `bg-surface-subtle`    | Subtle surface overlay                            |
| `bg-surface-medium`    | Medium surface overlay                            |
| `text-text`            | Primary text (near-white in dark mode)            |
| `text-text-secondary`  | Secondary text                                    |
| `text-text-muted`      | Muted / faint text                                |
| `text-accent`          | Brand accent (amber)                              |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EuroPython/website](https://github.com/EuroPython/website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
