---
trigger: always_on
description: > This document is primarily for AI agents and LLMs generating, reviewing, or modifying
---

# Creative Tim UI — Design & Usage Guide

**Version 1.0.0**
Creative Tim
March 2026

> **Note:**
> This document is primarily for AI agents and LLMs generating, reviewing, or modifying
> UI code that uses the Creative Tim UI block library. Guidance is optimised for
> automation and consistency. Humans may also find it useful as a reference.

---

## Abstract

Creative Tim UI is a block library of 390+ React + Tailwind CSS components built on shadcn/ui.
This guide covers the design philosophy behind every block, installation patterns, brand identity
rules, Tailwind v4 constraints, and component conventions so AI-generated code stays consistent
with the library — not just visually, but in the decisions it makes.

---

## Table of Contents

0. [Design Philosophy](#0-design-philosophy)
1. [Installing Blocks](#1-installing-blocks)
2. [PRO Blocks & API Key](#2-pro-blocks--api-key)
3. [Brand Identity](#3-brand-identity)
4. [Typography Rules](#4-typography-rules)
5. [Tailwind v4 Constraints](#5-tailwind-v4-constraints)
6. [Component Patterns](#6-component-patterns)
7. [Hydration Safety](#7-hydration-safety)
8. [Block Authoring Guide](#8-block-authoring-guide)

---

## 0. Design Philosophy

Across 300+ products and categories, Creative Tim has applied the same discipline: identify what developers actually need, build that exceptionally well, and cut everything else.

That discipline — research first, build only what's necessary, remove rather than add — is the foundation of every block in this library.

**The five principles:**

### 0.1 The 95% Rule

Build the common case exceptionally well. Do not reach for complex solutions to simple problems. A developer dropping in a block should not unwrap three layers of abstraction to change a label.

### 0.2 The "Light" Principle

A block is done not when there's nothing left to add, but when there's nothing left to remove. No feature flags for things that can be changed in code. No helper components used only once. No animations on elements that don't need to communicate state. No extra wrappers that exist only to satisfy a mental model.

### 0.3 Research Before Rendering

The first question is always: what problem does this solve for the person looking at the screen? Not what looks good in isolation. A pricing table reduces friction in a purchase decision. A dashboard surfaces the one number a person needs to act on. If the purpose isn't clear, the block isn't clear.

### 0.4 Standing on Foundations

shadcn/ui's `Table`, `Card`, `Button`, `Badge`, `Dialog` are well-designed, accessible, and maintained. Do not rebuild them. Do not wrap them unnecessarily. The Creative Tim contribution is the **composition** — how primitives are assembled into something useful for real work.

### 0.5 Real Use, Not Demo Use

The standard is: would a real team ship this to solve a real problem? Avoid visual complexity that doesn't carry information. A status badge that turns red when something is wrong is communication. A gradient that doesn't indicate anything is decoration. Only the first belongs.

Full detail with code examples: `rules/brand-philosophy.md`

---

---

## 1. Installing Blocks

### 1.1 Creative Tim CLI

The preferred method. Installs the block and all shadcn/ui primitive dependencies automatically.

```bash
# Initialize project (auto-detects Next.js / Vite / Remix / Astro)
npx @creative-tim/ui@latest init

# Add a block
npx @creative-tim/ui@latest add <block-name>
```

**Package:** `@creative-tim/ui` (v0.4.x+)
**Binaries:** `creative-tim-ui`, `creative-tim` (aliases for the same CLI)

```bash
# All of these are equivalent:
npx @creative-tim/ui@latest add hero-01
npx creative-tim-ui add hero-01
npx creative-tim add hero-01
```

**Examples:**
```bash
npx @creative-tim/ui@latest add hero-01
npx @creative-tim/ui@latest add ai-agent-activity-01
npx @creative-tim/ui@latest add agent-management-list-01
npx @creative-tim/ui@latest add pricing-01
```

The CLI copies the component into `components/ui/` (or your configured components directory) and runs `npx shadcn@latest add` for each dependency.

### 1.2 shadcn CLI (Full URL)

Use this when you prefer the shadcn workflow or when integrating with shadcn's registry tooling.

```bash
npx shadcn@latest add https://www.creative-tim.com/ui/r/<block-name>.json
```

**Examples:**
```bash
npx shadcn@latest add https://www.creative-tim.com/ui/r/hero-01.json
npx shadcn@latest add https://www.creative-tim.com/ui/r/testimonials-03.json
```

### 1.3 Block Discovery

Browse all blocks:
- Website: https://www.creative-tim.com/ui
- AI index: https://www.creative-tim.com/ui/llms.txt (grouped by category, machine-readable)

Individual block JSON (for programmatic access):
```
https://www.creative-tim.com/ui/r/<block-name>.json
```

---

## 2. PRO Blocks & API Key

PRO blocks require a `Creative-Tim-Api-Key` header when fetching the block JSON. Obtain a key at https://www.creative-tim.com/ui.

### 2.1 Environment Setup

```bash
# .env.local (never commit this)
CREATIVE_TIM_UI_API_KEY=pk_live_xxxxxxxxxxxxxxxx
```

### 2.2 CLI Install with Key

```bash
npx @creative-tim/ui@latest add <pro-block-name> --api-key $CREATIVE_TIM_UI_API_KEY
```

### 2.3 Global Login (persists across sessions)

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [creativetimofficial/ui](https://github.com/creativetimofficial/ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
