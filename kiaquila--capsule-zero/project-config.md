---
trigger: always_on
description: > Universal onboarding document for any AI agent (Claude Code, Codex, Gemini CLI, Cursor, etc.)
---

# AGENTS.md — Capsule Zero Onboarding

> Universal onboarding document for any AI agent (Claude Code, Codex, Gemini CLI, Cursor, etc.)

## What Is Capsule Zero?

**Capsule Zero** is a premium fashion-tech platform — "the Aesop of wardrobe apps". It helps affluent users (25–40 yo) build maximally productive capsule wardrobes using a proprietary color and wardrobe methodology. Core metric: **Outfit Productivity Ratio** (outfits / items).

**Tech stack:** Next.js 14+ App Router, React, TypeScript, Tailwind CSS v4, Flutter mobile app (iOS + Android), Supabase backend
**Languages:** EN (primary) and RU are active in MVP v1 — i18n from Day 1. ES-AR is globally deferred to MVP v2.
**Target:** Buenos Aires-based startup, global premium segment

## Current Phase & Status

| Phase                         | Status                                                                                                                                                                             |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0. Founder Vision             | COMPLETE — `.specify/memory/constitution.md`                                                                                                                                       |
| 1. Market Research            | COMPLETE — `docs_capsule_zero/marketing/go-to-market.md`                                                                                                                           |
| 2. Product Definition         | COMPLETE — `.specify/specs/001-capsule-zero-mvp/spec.md`, `docs_capsule_zero/project/methodology/`, `docs_capsule_zero/ux/emotion-map.md`, `docs_capsule_zero/ux/ux-validation.md` |
| 3. UX/UI Design               | COMPLETE — 16 logical screens across 12 HTML files + `html-prototypes/design-system.html`, `html-prototypes/color-system.html` — all in `html-prototypes/`                         |
| **4. Technical Architecture** | **DECISIONS DOCUMENTED** — mock-first Stage 1 posture accepted; integration gates pending before real provider flows                                                               |
| 5. Development Sprint         | Upcoming                                                                                                                                                                           |
| 6. QA & Soft Launch           | Upcoming                                                                                                                                                                           |
| 7. Commercial Launch          | Upcoming                                                                                                                                                                           |

**Locale scope decision, 2026-06-07:** Spanish / ES-AR is removed from active MVP v1 scope and moved globally to MVP v2. Keep Spanish source copy as future reference only; do not expose ES-AR in active routing, language switchers, profile language persistence, OpenAPI enums, generated clients, or launch acceptance criteria until the MVP v2 locale scope is reopened.

## Where to Find Specifications

```
.specify/
  memory/
    constitution.md      ← Project principles, methodology, design rules (READ FIRST)
    design-system.md     ← Glass tokens, colors, typography, components
    market-context.md    ← Competitors, persona, market size, pricing
  specs/
    001-capsule-zero-mvp/
      spec.md            ← Full MVP spec: 25 user stories, flows, requirements
      prototype-map.md   ← Maps HTML files → spec sections → screens
```

## HTML Prototypes

Located in `html-prototypes/`. These are **pixel-perfect hi-fi prototypes** (pure HTML+CSS, no frameworks) representing the approved Phase 3 design. The folder also contains the design system and color palette references used for development.

**Current source of truth:** the HTML prototypes in `html-prototypes/` are the most up-to-date product reference for product behavior, layout, and scope. If an older doc conflicts with an approved HTML prototype, follow the prototype and then align the docs.

| File                                  | Screen                                       | User Stories           |
| ------------------------------------- | -------------------------------------------- | ---------------------- |
| `html-prototypes/index.html`          | Landing + Auth popup                         | US-001, US-002, US-003 |
| `html-prototypes/auth.html`           | Standalone Auth                              | US-002, US-003         |
| `html-prototypes/dashboard.html`      | Dashboard                                    | US-004, US-005         |
| `html-prototypes/guided-journey.html` | Guided Journey (3 steps)                     | US-008–012, US-017     |
| `html-prototypes/capsule-result.html` | Capsule Result                               | US-013–016             |
| `html-prototypes/my-items.html`       | My Items                                     | US-006, US-007         |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kiaquila/capsule-zero](https://github.com/kiaquila/capsule-zero) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
