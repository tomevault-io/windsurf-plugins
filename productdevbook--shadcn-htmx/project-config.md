---
trigger: always_on
description: This repository builds a **shadcn-style component library for htmx v4 + Tailwind CSS v4**, anchored to **web standards**. The goal is components that work because the platform supports them — no hacks, no userland reinvention of features the browser already ships.
---

# Agent Guide

This repository builds a **shadcn-style component library for htmx v4 + Tailwind CSS v4**, anchored to **web standards**. The goal is components that work because the platform supports them — no hacks, no userland reinvention of features the browser already ships.

To make that possible, this repo vendors the source code and documentation of the libraries and specs we depend on under `repos/`. **You should read those vendored sources before you write or modify code.** They are not training data; they are the current, authoritative ground truth for this project.

---

## Hard rules

1. **`repos/` is read-only reference material.** Never import from it, bundle it, or copy code out of it into `src/`. If you need something from a vendored repo in our shipped code, rewrite it in our style — don't transplant.
2. **Do not commit changes inside `repos/`.** It is managed by `scripts/sync-repos.sh`. Local edits there will be wiped on the next sync.
3. **Prefer reading `repos/` over recalling from training data.** Your training cutoff predates current htmx v4 and Tailwind v4 details. When the vendored source disagrees with what you remember, the vendored source wins.
4. **No hacks.** If a feature requires polyfills, runtime feature-detection branches, or workarounds for "browsers don't do this yet," stop and ask. This project ships only what the platform supports natively today. Progressive enhancement is fine; emulation is not.
5. **Cite what you read.** When you make a non-obvious decision, mention which file under `repos/` justified it (e.g., `repos/htmx/src/htmx.js:1234`). This lets reviewers verify against the source.

---

## What's vendored, and when to read each

### `repos/htmx/` — htmx v4 source (branch: `four-dev`)

The htmx v4 development branch. **This is the version we target.** v3 syntax and behaviors are not authoritative here.

- **Read for:** attribute names and semantics (`hx-*`), event names, the request/swap lifecycle, extension API, behavioral differences from v3.
- **Best entry points:**
  - `repos/htmx/src/` — core implementation. `htmx.js` is the main file.
  - `repos/htmx/www/src/content/` — official docs (Astro content). Markdown reference for every attribute and concept.
  - `repos/htmx/www/reference.md` — single-file API reference.
  - `repos/htmx/CHANGELOG.md` — what changed from v3 to v4.
  - `repos/htmx/test/` — behavioral specifications by example.
- **Do not assume v3 behavior.** If you're not sure whether something changed in v4, grep the source.

### `repos/tailwindcss/` — Tailwind CSS v4 source (branch: `main`)

Tailwind v4 with the Oxide engine. CSS-first config (`@theme`, `@layer`, no `tailwind.config.js` by default), new variant syntax.

- **Read for:** v4 directives, default theme tokens, how `@theme` and `@utility` work, plugin authoring, container queries, the new color system.
- **Best entry points:**
  - `repos/tailwindcss/packages/tailwindcss/` — core engine.
  - `repos/tailwindcss/packages/tailwindcss/src/` — implementation of `@theme`, `@apply`, variants.
  - `repos/tailwindcss/packages/tailwindcss/preflight.css` and `theme.css` — what ships by default.
- **Do not write Tailwind v3 config files** (`tailwind.config.js` with `content:` arrays). Use the v4 CSS-first approach.

### `repos/shadcn-ui/` — shadcn/ui source (branch: `main`)

The original React/Radix-based shadcn. **We do not copy its code** (different stack), but we mirror its philosophy and component anatomy.

- **Read for:** which components a "shadcn-style library" should ship, what each component's API surface looks like, accessibility considerations baked into Radix, naming conventions, documentation structure.
- **Best entry points:**
  - `repos/shadcn-ui/apps/v4/registry/` — current v4 component registry. Source of truth for what to build and how to structure each component.
  - `repos/shadcn-ui/apps/v4/content/docs/components/` — per-component docs. Mirror this structure for our docs.
  - `repos/shadcn-ui/packages/shadcn/` — the CLI. Useful if we ever ship our own.
- **Do not copy React code into our htmx components.** Read for *intent and anatomy*, then translate to htmx + server-rendered HTML.

### `repos/mdn/` — MDN Web Docs (trimmed to `web/html`, `web/css`, `web/accessibility`, `web/api`)

The reference for what the platform actually does. Trimmed to the parts that matter for us.

- **Read for:** semantic HTML element behavior, ARIA roles and properties, CSS property/selector specifics, DOM and Web API contracts.
- **Best entry points:**
  - `repos/mdn/files/en-us/web/html/reference/elements/` — every HTML element.
  - `repos/mdn/files/en-us/web/css/` — every CSS property, selector, and at-rule.
  - `repos/mdn/files/en-us/web/accessibility/` — ARIA, WCAG, accessible UI patterns.
  - `repos/mdn/files/en-us/web/api/` — DOM, Fetch, Web Components, etc.
- **When in doubt about a native element or attribute, check MDN before reaching for JS or a Tailwind plugin.**

### `repos/web.dev/` — Google web.dev articles and Learn courses


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [productdevbook/shadcn-htmx](https://github.com/productdevbook/shadcn-htmx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
