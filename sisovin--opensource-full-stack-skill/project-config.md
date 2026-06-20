---
trigger: always_on
description: |
---


# Full-Stack UI/UX + PHP Skill

This repository contains two major capability sets:

1. UI/UX intelligence driven by searchable design datasets and generation scripts.
2. Native PHP full-stack engineering guidance for production-grade apps.

This file consolidates both into one operating guide.

## When to apply

Use this skill whenever the task changes how the product looks, behaves, or works.

### Must use

- New screens: landing pages, dashboards, admin panels, SaaS views, app interfaces.
- UI component work: buttons, cards, forms, tables, dialogs, navigation, charts.
- Design decisions: style, palette, typography, spacing, interaction, motion.
- Frontend implementation: HTML5, Tailwind CSS v4, vanilla JS patterns.
- Backend implementation in PHP: routing, MVC, services, repositories, APIs.
- Data/auth/security work: PDO queries, transactions, sessions, CSRF, JWT, RBAC.
- Full-stack feature delivery that crosses UI and backend boundaries.

### Recommended

- UI looks inconsistent or low-quality but root cause is unclear.
- Users report usability, accessibility, or interaction friction.
- Auth flows need hardening, cleanup, or modernization.
- You need consistent rules for both design and implementation.

### Skip

- Pure infrastructure or DevOps tasks with no product/UI/backend code changes.
- Tasks unrelated to web app behavior.

## Core operating model

### Track A: UI/UX intelligence

Use data-backed search and design-system generation to pick style direction,
colors, typography, UX rules, and stack-specific guidance.

### Track B: PHP full-stack engineering

Use the references for architecture, secure auth, authorization, and MySQL/PDO
implementation details in a framework-free PHP 8.5 codebase.

### Track C: Integrated execution

For real features, run both tracks in order:

1. Define UX and visual rules.
2. Implement frontend structure and interactions.
3. Build backend routes, services, and persistence.
4. Apply security and access controls.
5. Verify quality with UX + security checklists.

## Rule categories by priority (long-form)

Use this matrix to decide validation order before implementation and before release.

| Priority | Category | Impact | Domain | Key checks (must have) | Anti-patterns (avoid) |
|---|---|---|---|---|---|
| 1 | Accessibility | CRITICAL | ux | Contrast 4.5:1, alt text, keyboard nav, aria labels | Hidden focus, icon-only controls without labels |
| 2 | Touch and interaction | CRITICAL | ux | Min target 44x44, 8px spacing, loading feedback | Hover-only behavior, instant unannounced state jumps |
| 3 | Performance | HIGH | ux | WebP/AVIF, lazy loading, reserve layout space | Layout thrashing, cumulative layout shift |
| 4 | Style selection | HIGH | style, product | Product-fit style, consistency, SVG icons | Random style mixing, emoji-as-icon structure |
| 5 | Layout and responsive | HIGH | ux | Mobile-first, viewport meta, no horizontal scroll | Fixed narrow desktop assumptions on mobile |
| 6 | Typography and color | MEDIUM | typography, color | 16px base, line-height 1.5+, semantic tokens | Tiny body text, gray-on-gray, raw ad-hoc hex values |
| 7 | Animation | MEDIUM | ux | 150-300ms timing, meaningful motion, continuity | Decorative-only motion, width/height animation |
| 8 | Forms and feedback | MEDIUM | ux | Visible labels, inline errors, helper text | Placeholder-only labels, delayed ambiguous errors |
| 9 | Navigation patterns | HIGH | ux | Predictable back, max 5 bottom tabs, deep links | Overloaded nav, broken back stack |
| 10 | Charts and data | LOW | chart | Correct chart type, legend/tooltips, accessible colors | Color-only encoding, unreadable dense charts |

## UI/UX quick reference (long-form)

### 1) Accessibility (CRITICAL)

- color-contrast: minimum 4.5:1 for normal text, 3:1 for large text.
- focus-states: keep visible focus indicators on interactive elements.
- alt-text: meaningful alt text for meaningful images.
- aria-labels: label icon-only controls.
- keyboard-nav: tab order must match visual order.
- form-labels: each input needs a visible label.
- skip-links: support skip to main content.
- heading-hierarchy: keep heading levels sequential.
- color-not-only: never use color as the only signal.
- dynamic-type: support text scaling without clipped UI.
- reduced-motion: honor reduced motion preferences.
- voiceover-sr: ensure logical screen-reader reading order.
- escape-routes: provide cancel/back in dialogs and flows.
- keyboard-shortcuts: preserve system and accessibility shortcuts.

### 2) Touch and interaction (CRITICAL)

- touch-target-size: minimum 44x44pt (iOS) or 48x48dp (Android).
- touch-spacing: minimum 8px spacing between targets.
- hover-vs-tap: primary interactions cannot depend on hover.
- loading-buttons: disable and show pending state during async.
- error-feedback: show local, actionable errors near source.
- cursor-pointer: use pointer cursor for clickable web elements.
- gesture-conflicts: avoid conflicting nested gestures.
- tap-delay: use touch-action where relevant to reduce delay.
- standard-gestures: follow platform-standard gestures.
- system-gestures: do not block OS-level gestures.
- press-feedback: give immediate visual press response.
- haptic-feedback: use haptics sparingly for key confirmations.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sisovin/OpenSource-Full-Stack-Skill](https://github.com/sisovin/OpenSource-Full-Stack-Skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
