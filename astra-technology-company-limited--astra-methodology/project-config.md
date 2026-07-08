---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**astra-methodology** is a Claude Code plugin that implements the ASTRA (AI-augmented Sprint Through Rapid Assembly) methodology. It provides Sprint 0 project initialization, coding convention enforcement (Java/TypeScript/React Native/Python/CSS/SCSS), Korean public data standard enforcement, international code standards (ISO 3166-1/2, ITU-T E.164), naming validation, and quality gates for Korean enterprise software development.

This is NOT an application codebase — it is a Claude Code plugin consisting of skills, agents, hooks, commands, and scripts that get installed into target projects.

## Repository Structure

```
astra-methodology/
├── skills/              # Claude Code skills (each subdir has SKILL.md with full details)
├── agents/              # Specialized subagents (read-only, *-validator/*-reviewer/*-runner/*-analyzer/*-persona)
├── commands/            # Slash commands (lighter than skills)
├── hooks/               # PostToolUse hooks (hooks.json)
├── scripts/             # Shell scripts for hooks and verification
├── data/                # Standard dictionary + ISO/ITU code JSON files (large — use jq queries)
├── docs/                # Reference design/UX/dev guides (ux/, catalog/, manual/, plugin/, development/)
└── .claude-plugin/      # Plugin manifest (plugin.json, marketplace.json)
```

For per-skill details, read each `skills/{name}/SKILL.md`. For per-agent capabilities, read each `agents/{name}.md`. For full data file inventory, see `data/`.

## Key Concepts

### ASTRA Methodology

- **VIP Principles**: Vibe-driven Development, Instant Feedback Loop, Plugin-powered Quality
- **Sprint cycle**: 1 week
- **Team roles**: VA (Vibe Architect), PE (Prompt Engineer), DE (Domain Expert), DSA (Design System Architect)
- **Quality Gates**: Gate 1 (write-time/automatic), Gate 2 (review-time), Gate 2.5 (design review), Gate 3 (release-time)

### Korean Public Data Standard (행정안전부 공공데이터 공통표준)

The plugin enforces naming conventions from the Korean Ministry of the Interior and Safety's public data standard dictionary. Key rules:

- **Table prefixes**: `TB_` (general), `TC_` (code), `TH_` (history), `TL_` (log), `TR_` (relation)
- **Column suffixes**: `_YMD` (date), `_DT` (datetime), `_AMT` (amount), `_NM` (name), `_CD` (code), `_NO` (number), `_CN` (content), `_YN` (yes/no), `_SN` (sequence), `_ADDR` (address)
- **Forbidden words**: `standard_words.json` contains a `금칙어목록` field; violations trigger warnings with standard alternatives

### Coding Convention Enforcement

The plugin auto-applies coding conventions when editing language-specific files:

- **Java** (Google Java Style Guide): 2-space indent, 100-char limit, K&R braces, no wildcard imports, `UpperCamelCase` classes, `lowerCamelCase` methods, `UPPER_SNAKE_CASE` constants
- **TypeScript** (Google TypeScript Style Guide): Prettier formatting, no `export default`, no `any`, no `var`, no `.forEach()`, `===`/`!==` required, named exports only
- **React Native** (Airbnb React/JSX + Obytes RN Starter + React Native Official): Complementary layer on TypeScript convention for RN/Expo projects. `kebab-case` files, functional components only, `PascalCase` components, `StyleSheet.create()` or NativeWind, TanStack Query + Zustand, Expo Router, max 3 params/110 lines per function, no inline styles, no class components
- **Python** (PEP 8): 4-space indent, 79-char limit, `snake_case` functions, `CapWords` classes, `is None` required, no bare `except:`
- **CSS/SCSS** (CSS Guidelines + Sass Guidelines): 2-space indent, 80-char limit, BEM naming, no ID selectors, max 3-level nesting, mobile-first media queries

Reference files are in `skills/coding-convention/` (e.g., `java-coding-convention.md`, `typescript-coding-convention.md`, `react-native-coding-convention.md`). For mobile projects, the coding convention skill additionally references `docs/ux/mobile-design-guide.md`.

### Vibe Coding Design & Animation Guides

The plugin provides comprehensive design and animation guides under `docs/ux/` that should be referenced during all UI design and implementation work:

- **`vibe-coding-design-guide.md`**: anti-AI aesthetics prompting, reference-anchored design, constraint-first approach, design token injection, tool comparison, DO/DON'T patterns
- **`vibe-coding-animation-guide.md`**: CSS native (View Transitions API, Scroll-Driven Animations, `@starting-style`, `linear()` springs), Framer Motion/GSAP/Lottie/Rive, micro-interactions, performance, 3-tier motion accessibility, Disney 12 principles

These guides are automatically loaded by `/service-planner` (Step 6 HTML mockup generation) and should be referenced by any skill or workflow that involves UI design, design system work, or animation implementation.

### International Code Standards (ISO 3166-1/2, ITU-T E.164)

The plugin auto-applies international code standards when implementing phone number inputs, country/region selectors, and address forms:

- **ISO 3166-1**: alpha-2 country codes (e.g., `KR`, `US`, `JP`) — stored as `NATN_CD CHAR(2)`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ASTRA-TECHNOLOGY-COMPANY-LIMITED/astra-methodology](https://github.com/ASTRA-TECHNOLOGY-COMPANY-LIMITED/astra-methodology) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
