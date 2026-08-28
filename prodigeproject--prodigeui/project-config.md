---
trigger: always_on
description: > ⚠️ **Read `PHILOSOPHY.md` first.** ProdigeUI has two modes. In **Creative Mode** (vague
---

# ProdigeUI — Agent Entry Point

> ⚠️ **Read `PHILOSOPHY.md` first.** ProdigeUI has two modes. In **Creative Mode** (vague
> brief) you ARE the designer — produce cinematic, craft-rich output using the `craft/`
> library; it must look visibly BETTER than raw AI, never merely "safe." In **Enhancement
> Mode** (specific brief) honor the designer's intent, execute their chosen techniques at
> reference quality, and DO NO HARM (never break assets, slow interaction, or add jank).
>
> ⚠️ **Rules prevent bad output; craft produces great output. You need BOTH.** The tokens,
> rules, and anti-slop gate below are the floor. For any expressive build, start at
> `craft/AGENTS.md` and pick a signature — otherwise output will be competent but
> forgettable, which is a FAILURE for expressive work.

> This document is the canonical entry point for AI agents using ProdigeUI.
> Read this file first. It tells you what ProdigeUI is, where things live, and how to work.

## System Purpose

ProdigeUI is a **portable UI/UX knowledge system** that equips you (AI agent) with:

- Measurable design rules backed by theory and research (80+ books, 39 repos — 119 sources indexed in `research/research-log.json`)
- A three-layer token system as the single source of visual truth
- Enterprise-grade component specifications (states, accessibility, variants)
- Accessible and purposeful motion presets with reduce-motion compliance
- Professional prompt templates for 6 use-case categories
- A quality gate ensuring output is free of "AI slop"
- The **Three Dials** system for calibrating aesthetic direction per project

Use these artifacts as the foundation for every visual decision. Define concrete values once
at a token boundary; repeated component styling must consume semantic variables rather than
repeating raw literals.

---

## Available Skills

Skills are structured capability units stored in `skills/`. Each has a `SKILL.md` with
frontmatter (name, description, triggers) and step-by-step workflow instructions.

| Skill | Triggers | Purpose |
|-------|----------|---------|
| **prodige-ui-end-to-end** | "design ui", "create interface", "build component", "ui end to end", "design from brief" | Full workflow from brief to implementation with Quality Gate validation |
| **quality-check** | "check quality", "run quality gate", "audit design", "anti slop check" | Evaluate output against criteria.json and anti-AI-slop checklist |
| **token-management** | "manage tokens", "add token", "update token", "validate tokens" | Add, modify, and validate tokens across primitive/semantic/component layers |
| **theme-creation** | "create theme", "new theme", "custom theme", "brand theme" | Create a new theme with palette selection, token mapping, and contrast verification |
| **motion-review** | "review animations", "review motion", "check animations" | Review motion code against a high craft bar (easing, frequency, origin, interruptibility, GPU, a11y) |
| **design-lens** | "make it bolder", "tone it down", "fix the spacing", "colors feel flat", "polish this" | Apply a focused adjustment lens to existing output instead of rebuilding (`craft/design-lenses.md`) |

# ProdigeUI — Agent Entry Point

> ⚠️ **Read `PHILOSOPHY.md` first.** ProdigeUI has two modes. In **Creative Mode** (vague
> brief) you ARE the designer — produce cinematic, craft-rich output using the `craft/`
> library; it must look visibly BETTER than raw AI, never merely "safe." In **Enhancement
> Mode** (specific brief) honor the designer's intent, execute their chosen techniques at
> reference quality, and DO NO HARM (never break assets, slow interaction, or add jank).
>
> ⚠️ **Rules prevent bad output; craft produces great output. You need BOTH.** The tokens,
> rules, and anti-slop gate below are the floor. For any expressive build, start at
> `craft/AGENTS.md` and pick a signature — otherwise output will be competent but
> forgettable, which is a FAILURE for expressive work.

> This document is the canonical entry point for AI agents using ProdigeUI.
> Read this file first. It tells you what ProdigeUI is, where things live, and how to work.

## System Purpose

ProdigeUI is a **Generative UI/UX Knowledge Engine** that equips AI agents to synthesize world-class interfaces driven by 4 core pillars:

1. **Intent & Product Read**: Deeply analyzing product domain, user persona, and emotional positioning.
2. **Dynamic Taste Engine**: Deriving custom HSL color harmonies (`themes/generative-theme-synthesis.md`), font pairings, material depth, and spatial rhythms dynamically for ANY brief.
3. **High-Craft Copywriting System**: Writing authentic domain storytelling copy (`craft/high-craft-copywriting.md`) while strictly prohibiting prompt instruction leaks (e.g. `(CLICK TO...)`).
4. **Non-Negotiable Guardrails**: Enforcing WCAG AA contrast (>=4.5:1 / 7.0:1+), 0 inline HTML styles, `prefers-reduced-motion`, and strict container boundary containment (`overflow: hidden;`).

---

## Available Skills

Skills are structured capability units stored in `skills/`. Each has a `SKILL.md` with
frontmatter (name, description, triggers) and step-by-step workflow instructions.

| Skill | Triggers | Purpose |
|-------|----------|---------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prodigeproject/prodigeui](https://github.com/prodigeproject/prodigeui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
