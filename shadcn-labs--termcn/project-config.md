---
trigger: always_on
description: > This document is mainly for agents and LLMs to follow when implementing
---

# React View Transitions

**Version 1.0.0**
Vercel Engineering
March 2026

> **Note:**
> This document is mainly for agents and LLMs to follow when implementing
> view transitions in React applications. Humans may also find it useful,
> but guidance here is optimized for automation and consistency by
> AI-assisted workflows.

---

## Abstract

Guide for implementing smooth, native-feeling animations using React's View Transition API. Covers the `<ViewTransition>` component, `addTransitionType`, CSS view transition pseudo-elements, shared element transitions, Suspense reveals, list reorder, directional navigation, and Next.js integration. Includes a step-by-step implementation workflow, ready-to-use CSS animation recipes, and common mistake warnings.

---

## Table of Contents

1. [Core Reference](#when-to-animate)
   - [When to Animate](#when-to-animate)
   - [Availability](#availability)
   - [Core Concepts](#core-concepts)
   - [Styling with View Transition Classes](#styling-with-view-transition-classes)
   - [Transition Types](#transition-types)
   - [Shared Element Transitions](#shared-element-transitions)
   - [Common Patterns](#common-patterns)
   - [How Multiple VTs Interact](#how-multiple-vts-interact)
   - [Next.js Integration](#nextjs-integration)
   - [Accessibility](#accessibility)
2. [Implementation Workflow](#implementation-workflow)
   - [Step 1: Audit the App](#step-1-audit-the-app)
   - [Step 2: Add CSS Recipes](#step-2-add-css-recipes)
   - [Step 3: Isolate Persistent Elements](#step-3-isolate-persistent-elements)
   - [Step 4: Add Directional Page Transitions](#step-4-add-directional-page-transitions)
   - [Step 5: Add Suspense Reveals](#step-5-add-suspense-reveals)
   - [Step 6: Add Shared Element Transitions](#step-6-add-shared-element-transitions)
   - [Step 7: Verify Each Navigation Path](#step-7-verify-each-navigation-path)
   - [Common Mistakes](#common-mistakes)
3. [Patterns and Guidelines](#patterns-and-guidelines)
4. [CSS Animation Recipes](#css-animation-recipes)
5. [View Transitions in Next.js](#view-transitions-in-nextjs)

---

Animate between UI states using the browser's native `document.startViewTransition`. Declare *what* with `<ViewTransition>`, trigger *when* with `startTransition` / `useDeferredValue` / `Suspense`, control *how* with CSS classes. Unsupported browsers skip animations gracefully.

## When to Animate

Every `<ViewTransition>` should communicate a spatial relationship or continuity. If you can't articulate what it communicates, don't add it.

Implement **all** applicable patterns from this list, in this order:

| Priority | Pattern | What it communicates |
|----------|---------|---------------------|
| 1 | **Shared element** (`name`) | "Same thing — going deeper" |
| 2 | **Suspense reveal** | "Data loaded" |
| 3 | **List identity** (per-item `key`) | "Same items, new arrangement" |
| 4 | **State change** (`enter`/`exit`) | "Something appeared/disappeared" |
| 5 | **Route change** (layout-level) | "Going to a new place" |

This is an implementation order, not a "pick one" list. Implement every pattern that fits the app. Only skip a pattern if the app has no use case for it.

### Choosing Animation Style

| Context | Animation | Why |
|---------|-----------|-----|
| Hierarchical navigation (list → detail) | Type-keyed `nav-forward` / `nav-back` | Communicates spatial depth |
| Lateral navigation (tab-to-tab) | Bare `<ViewTransition>` (fade) or `default="none"` | No depth to communicate |
| Suspense reveal | `enter`/`exit` string props | Content arriving |
| Revalidation / background refresh | `default="none"` | Silent — no animation needed |

Reserve directional slides for hierarchical navigation (list → detail) and ordered sequences (prev/next photo, carousel, paginated results). For ordered sequences, the direction communicates position: "next" slides from right, "previous" from left. Lateral/unordered navigation (tab-to-tab) should not use directional slides — it falsely implies spatial depth.

---

## Availability

- **Next.js:** Do **not** install `react@canary` — the App Router already bundles React canary internally. `ViewTransition` works out of the box. `npm ls react` may show a stable-looking version; this is expected.
- **Without Next.js:** Install `react@canary react-dom@canary` (`ViewTransition` is not in stable React).
- Browser support: Chromium 111+, Firefox 144+, Safari 18.2+. Graceful degradation.

---

## Core Concepts

### The `<ViewTransition>` Component

```jsx
import { ViewTransition } from 'react';

<ViewTransition>
  <Component />
</ViewTransition>
```

React auto-assigns a unique `view-transition-name` and calls `document.startViewTransition` behind the scenes. Never call `startViewTransition` yourself.

### Animation Triggers

| Trigger | When it fires |
|---------|--------------|
| **enter** | VT first inserted during a Transition |
| **exit** | VT first removed during a Transition |
| **update** | DOM mutations inside a VT. With nested VTs, mutation applies to the innermost one |
| **share** | Named VT unmounts and another with same `name` mounts in same Transition |

Only `startTransition`, `useDeferredValue`, or `Suspense` activate VTs. Regular `setState` does not animate.

### Critical Placement Rule


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shadcn-labs/termcn](https://github.com/shadcn-labs/termcn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
