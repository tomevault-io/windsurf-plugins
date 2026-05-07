---
trigger: always_on
description: Clean, professional, fast. No gimmicks. The interface should feel like a native tool, not a flashy website.
---

# Design System & Development Guidelines

## Design Philosophy

Clean, professional, fast. No gimmicks. The interface should feel like a native tool, not a flashy website.

**Respect the user**: No cookie popups, no notifications/badges, no onboarding tours, no update prompts, no newsletter, no AI, no review begging. Request permissions only when the feature is used.

## Product Vision

This repository is no longer treated as a single-purpose deleted-messages utility. The product direction is a modular Telegram tooling workspace built on a shared multi-account auth/session platform.

- `Scheduled Messages` and `LLM Context Export` are first-class modules, not lab-only features.
- `LLM Context Export` is an export/formatting workflow for external assistants, not an in-app AI product.
- Productionization work should simplify and harden the platform without collapsing the module system.

## Documentation Map

- `TODO.md` is the restart brief and priority order for future work.
- `README.md` is the public product/development overview.
- `AGENTS.md` is the single agent-facing guide for repo rules, architecture notes, and product direction.
- If docs and current code disagree, treat the codebase as the source of current behavior and `TODO.md` as the source of target direction.

## Documentation Durability

- Long-lived docs in this repo should capture durable guidance: product direction, architecture, operating rules, recurring pitfalls, and priority order.
- Do not put point-in-time coordination data in `AGENTS.md`, `TODO.md`, or other durable docs. That includes PR numbers, branch names, commit SHAs, CI run IDs, "safe merge checkpoints", temporary statuses, and notes that only make sense right now.
- If a detail would read as strange after a few commits, a month, or to a different person, it does not belong in durable docs.
- Put ephemeral status in PR descriptions, issues, commit messages, or short-lived handoff comments instead.

## Returning Later

- Read `TODO.md` first, then this file, then `README.md`.
- Re-establish the baseline with the standard verification commands before changing behavior.
- Prioritize live Telegram validation and safe local-data lifecycle follow-through before major rewrites or visual refreshes.

## Border Radius

Use minimal rounding for a professional appearance:

| Element | Tailwind Class | Pixels |
|---------|---------------|--------|
| Buttons | `rounded-md` | 6px |
| Cards | `rounded-lg` | 8px |
| Inputs | `rounded` | 4px |
| Modals | `rounded-xl` | 12px |
| Pills/Badges | `rounded-full` | Full |
| Tooltips | `rounded` | 4px |

**Never use**: `rounded-2xl`, `rounded-3xl` for main UI elements.

## Shadows

Subtle, functional shadows only:

| Use Case | Tailwind Class |
|----------|---------------|
| Cards (resting) | `shadow-sm` |
| Cards (hover) | `shadow` |
| Modals | `shadow-lg` |
| Dropdowns | `shadow-lg` |
| Buttons | No shadow |

## Animations & Transitions

Instant feedback. Animations should be imperceptible, not decorative.

| Type | Duration | Easing |
|------|----------|--------|
| Hover effects | 100ms | ease-out |
| State changes | 100ms | ease-out |
| Modal open/close | 100ms | ease-out |
| Page transitions | 100ms | ease-out |
| Loading spinners | 600ms | linear |

```css
/* Standard transition */
transition: all 100ms ease-out;

/* Tailwind */
transition-all duration-100 ease-out
```

**Never use**: Slow transitions (>150ms for interactions), bouncy effects, or decorative animations.

## Color Palette

Telegram-inspired, with good dark mode support:

### Light Mode
- Background: `bg-gray-50` (#f9fafb)
- Card: `bg-white` (#ffffff)
- Text: `text-gray-900` (#111827)
- Muted: `text-gray-500` (#6b7280)
- Border: `border-gray-200` (#e5e7eb)

### Dark Mode
- Background: `dark:bg-gray-950` (#030712)
- Card: `dark:bg-gray-900` (#111827)
- Text: `dark:text-white` (#ffffff)
- Muted: `dark:text-gray-400` (#9ca3af)
- Border: `dark:border-gray-800` (#1f2937)

### Accent Colors
- Primary: `blue-600` (#2563eb)
- Primary Hover: `blue-700` (#1d4ed8)
- Success: `green-600` (#16a34a)
- Warning: `amber-500` (#f59e0b)
- Error: `red-600` (#dc2626)
- Bot Accent: `purple-600` (#9333ea)

## Typography

System fonts for performance, clear hierarchy:

```css
font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
```

| Element | Classes |
|---------|---------|
| Page Title | `text-2xl font-bold` |
| Section Title | `text-lg font-semibold` |
| Card Title | `text-base font-medium` |
| Body | `text-sm` |
| Caption | `text-xs text-gray-500` |

## Spacing

8px grid system:

- `p-1` = 4px
- `p-2` = 8px
- `p-3` = 12px
- `p-4` = 16px
- `p-6` = 24px
- `p-8` = 32px

Use consistent spacing within component types.

## Component Patterns

### Buttons

```html
<!-- Primary -->
<button class="px-4 py-2 rounded-md font-medium text-sm bg-blue-600 text-white hover:bg-blue-700 transition-colors duration-150">
  Action
</button>

<!-- Secondary -->
<button class="px-4 py-2 rounded-md font-medium text-sm bg-gray-100 dark:bg-gray-800 text-gray-700 dark:text-gray-300 hover:bg-gray-200 dark:hover:bg-gray-700 transition-colors duration-150">
  Cancel
</button>
```

### Cards

```html

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uburuntu/Telegram-Toolset](https://github.com/uburuntu/Telegram-Toolset) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
