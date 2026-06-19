---
trigger: always_on
description: Use this skill for any accessibility concern: WCAG compliance (any success criterion including SC 1.3.5 autocomplete, SC 2.4.11 focus obscured), ARIA roles/attributes, screen reader behavior, axe-core or WAVE audit failures, color/non-text contrast, keyboard navigation, focus trapping/management, skip links, touch targets, VPAT reports, or accessible form patterns. Also trigger when a user asks whether a UI library (shadcn, Radix) handles keyboard interactions — but only if the question is speci
---


# Accessibility

> **Standard:** WCAG 2.2 AA required · AAA aspirational
> **Stack:** React 19 · shadcn/ui · Tailwind CSS v4 · Radix UI (`radix-ui` v1.4.3)
> **POUR Principles:** Perceivable · Operable · Understandable · Robust

Implement WCAG 2.2 AA compliant accessibility in React 19 components using the draft_v0 stack. Items marked **(AAA)** exceed the required AA level and are aspirational best practices to pursue where feasible.

---

## POUR Reference Map

| Principle | Key Topics | Reference Files |
|-----------|-----------|-----------------|
| **Perceivable** | Alt text, landmarks, headings, data tables, color contrast, non-text contrast, text spacing, reflow | [semantic-html.md](references/semantic-html.md), [mobile-touch.md](references/mobile-touch.md) |
| **Operable** | Keyboard, focus management, focus not obscured, skip links, page titles, target size, dragging, pointer gestures | [keyboard-navigation.md](references/keyboard-navigation.md), [focus-management.md](references/focus-management.md), [mobile-touch.md](references/mobile-touch.md), [wcag22-new-criteria.md](references/wcag22-new-criteria.md) |
| **Understandable** | ARIA, forms, error handling, auth, redundant entry, consistent help, live regions | [aria-attributes.md](references/aria-attributes.md), [forms-a11y.md](references/forms-a11y.md) |
| **Robust** | Screen reader support, status messages, automated + manual testing, VPAT, CI/CD | [testing-a11y.md](references/testing-a11y.md) |

---

## WCAG 2.2 New Criteria — Enterprise Baseline

WCAG 2.2 added 9 success criteria not present in WCAG 2.1. All **AA** items are **required**; **AAA** items are aspirational.

| SC | Title | Level | Requirement in One Line |
|----|-------|-------|-------------------------|
| 2.4.11 | Focus Not Obscured (Minimum) | **AA** | Focused element not fully hidden behind sticky/fixed UI chrome |
| 2.4.12 | Focus Not Obscured (Enhanced) | AAA | Focused element not obscured at all — not even partially |
| 2.4.13 | Focus Appearance | AAA | Focus indicator ≥ 2px perimeter, 3:1 contrast ratio |
| 2.5.7 | Dragging Movements | **AA** | Every drag action has a single-pointer (click/tap) alternative |
| 2.5.8 | Target Size (Minimum) | **AA** | Interactive targets ≥ 24×24 CSS pixels |
| 3.2.6 | Consistent Help | **A** | Help mechanisms in same relative position across pages |
| 3.3.7 | Redundant Entry | **A** | Previously entered data auto-populated or selectable |
| 3.3.8 | Accessible Authentication (Minimum) | **AA** | No cognitive puzzle required in auth unless alternative exists |
| 3.3.9 | Accessible Authentication (Enhanced) | AAA | No cognitive test in any auth step, no exceptions |

See [wcag22-new-criteria.md](references/wcag22-new-criteria.md) for detailed implementation guidance, code examples, common failures, and test procedures for each criterion.

---

## Accessibility Utilities in This Project

| Tool / Pattern | How to Use | Notes |
|----------------|-----------|-------|
| `FocusScope` | `import { FocusScope } from 'radix-ui'` | Focus trapping for modals/dialogs. Already installed (v1.4.3). |
| `sr-only` | `className="sr-only"` | Visually hidden, screen-reader visible text. |
| `focus-visible:` | `focus-visible:ring-[3px] focus-visible:ring-ring/50` | Focus ring on keyboard interaction only (not mouse click). |
| `motion-reduce:` | `motion-reduce:transition-none` | Respects `prefers-reduced-motion` OS preference. |
| `aria-*:` | `aria-invalid:border-destructive` | Style elements based on ARIA state via Tailwind variant. |
| `scroll-mt-20` | `focus-visible:scroll-mt-20` | Scroll margin for Focus Not Obscured (2.4.11 AA). |
| `min-h-6 min-w-6` | `className="min-h-6 min-w-6"` | 24×24px minimum target size (2.5.8 AA). |
| `min-h-[44px] min-w-[44px]` | `className="min-h-[44px] min-w-[44px]"` | 44×44px recommended target size (2.5.5 AAA). |
| Custom hooks | `src/client/hooks/` (`@/hooks`) | `useKeyPress`, `useReducedMotion`, `useDocumentTitle`, `useAnnounce`. |

---

## shadcn/Radix Built-in Accessibility

**shadcn/ui components are accessible by default** — built on Radix UI primitives that handle ARIA roles, keyboard navigation, and focus management automatically. Do not re-implement what these components already provide.

| Component | What Radix Handles Automatically |
|-----------|----------------------------------|
| `Dialog` | `role="dialog"`, `aria-modal`, focus trap (`FocusScope`), `Escape` to close, return focus on close |
| `AlertDialog` | `role="alertdialog"`, focus trap, prevents accidental `Escape` dismissal |
| `DropdownMenu` | `role="menu"` / `menuitem`, roving tabindex, Arrow keys, `Escape` to close |
| `Select` | `role="listbox"` / `option`, Arrow keys, type-ahead search |
| `Tabs` | `role="tablist"` / `tab` / `tabpanel`, Arrow Left/Right navigation |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [michelve/accessibility](https://github.com/michelve/accessibility) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
