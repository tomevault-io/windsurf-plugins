---
trigger: always_on
description: Small, clear, safe steps — and one source of truth in `/ops`.
---

# AGENTS.md — Comfy with ComfyUI (Docs) / Agent Principles

## Motto

Small, clear, safe steps — and one source of truth in `/ops`.

> Any verbal agreement must be reflected in `/ops` immediately.

---

## Live Directives Trump Documentation

If a conflict exists between older documentation and a newer verbal or written directive from the owner, the **newer directive takes precedence**.

Once a decision is made, you **must update `/ops`** to prevent documentation drift from reality.

---

## 1. Purpose

The primary purpose is to provide a **systematic and reproducible** documentation set for ComfyUI and its workflows.

* Treat `/ops` as the **Single Source of Truth (SSOT)** for Information Architecture (IA), writing rules, design tokens, and contribution rules.
* Keep the implementation (11ty, layouts, components, content) **aligned with `/ops` at all times**.

---

## 2. Scope and Non-Scope

### In Scope

* The documentation site built with Eleventy (11ty).
    * **Canonical URL Pattern**: `/<lang>/<section>/<slug>/`
* IA structure (sections, page types, navigation) as defined in `/ops`.
* Content and examples for:
    * Local ComfyUI setup and usage.
    * AI capabilities (models, depth, video, etc.).
    * Basic and advanced workflows with JSON examples.
* Design system and UI behavior as defined in:
    * `/ops/style-design.md` (Visual and layout tokens)
    * `/ops/style-writing.md` (Writing rules, headings, examples)
* JavaScript written **exclusively as ES Modules**.
* SVG icons that **inherit `currentColor`**.

### Out of Scope

* Maintaining external project-management tools (Notion, Scrapbox, etc.).
* Supporting legacy themes, deprecated UI, or third-party skins not aligned with `/ops`.
* "Random experiments" that are not tied to a concrete document or Architecture Decision Record (ADR).

---

## 3. Ground Rules

### No Silent UX / UI Changes

Any UI or design change must be:

1.  **Proposed and agreed upon** with the human owner.
2.  Implemented as a **focused change** (branch + PR).
3.  **Not extended or "improved"** further without explicit agreement.

### `/ops` First Principle

If you change behavior, IA, naming, or conventions, you **must start by updating `/ops`**.

Code and content are then updated to match the changes in `/ops`.

### Persistent IDs

* IDs, slugs, and major URLs **must be stable**.
* Renames require an **ADR and navigation updates**, not ad-hoc edits.

### Reproducible Workflows Only

Workflow JSONs must be complete enough for a third party to **reproduce the results**.

### Authorship Boundary

The human owner is the author, not the agent. Do not perform edits that exceed light proofreading of the owner's text.

---

## 4. Information Architecture and Writing

Detailed rules for IA (headings, page templates, narrative structure, examples) are enforced in a separate file.

* **Canonical Reference**: `/ops/style-writing.md`
* This file enforces that the rules defined there are **treated as binding**.
* Do not invent custom heading patterns or page layouts: **always follow `/ops/style-writing.md`**.

---

## 5. Tagging

Tags are optional.

When tags are present, they must be:

* **Consistent** with the page slug and topic.
* **Specific** but not spammy.
* **Maximum 5 tags per page.**

For `ai-capabilities`, default is no tags unless explicitly requested by the owner and documented in `/ops`.

---

## 6. URL and Identity

### Canonical URL Format

* **Format**: `/<lang>/<section>/<slug>/`
* `lang` $\in$ { `ja`, `en`, `zh` }
* `section` $\in$ { `begin-with`, `ai-capabilities`, `basic-workflows`, `data-utilities`, `notes` }
* `slug` is a **kebab-case, permanent identifier**.

### Slug Rules

* No **duplicate (`section`, `slug`) pairs** across the site.
* No "temporary" slugs; treat slugs as **stable IDs**.
* Slug changes require: **navigation update + ADR**.

### Assets

* Mockups and illustrations reside under `src/assets/mock/` or other agreed subfolders.
* All visual tokens (colors, radii, shadows, spacing) are defined in `/ops/style-design.md`.

---

## 7. Design Tokens and Assets

Colors, typography, border radii, spacing, and shadows are defined in `/ops/style-design.md`.

Any change to design tokens:

1.  **Must be proposed in `/ops/style-design.md` first.**
2.  Then implemented in CSS / utility classes.

Layout or component variations must align with:

* Existing token names.
* Previously agreed **responsive behavior**.

---

## 8. JavaScript / ESM

All JavaScript must be written as **ESM**:

* No new CommonJS modules.
* Use `"type": "module"` where appropriate.

### Icons

* Stored in `src/assets/icons/*.svg`.
* Must define a `viewBox`.
* Must use `fill="currentColor"` or **inherit `currentColor`**.

---

## 9. CI / Quality Gates

Before merging any Pull Request (PR):

* `section` + `slug` **uniqueness is preserved**.
* `nav.*.yml` slugs **match the actual file structure and IA**.

**Each page** must:

* Use the correct section and template as per `/ops`.
* If tags are present, keep them to $\leq 5$.

**All images** must:

* Use appropriate formats and sizes (no unbounded raw dumps).
* Provide explicit width/height unless rendered via the Gyazo pipeline (`{gyazo=image|loop}` or equivalent shortcode path) where layout sizing is enforced by renderer/CSS tokens.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nomadoor/Comfy-with-ComfyUI](https://github.com/nomadoor/Comfy-with-ComfyUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
