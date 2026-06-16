---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Tour Kit is a headless onboarding and product tour library for React. It's designed to work well with shadcn/ui and similar component libraries.

## Commands

Both `pnpm` and `bun` are supported. Use whichever you prefer.

```bash
# Install dependencies
pnpm install   # or: bun install

# Build all packages
pnpm build     # or: bun run build

# Run development mode (watch for changes)
pnpm dev       # or: bun run dev

# Type checking
pnpm typecheck # or: bun run typecheck

# Run single package commands (via turbo)
pnpm build --filter=@tour-kit/core   # or: bun run build --filter=@tour-kit/core
```

## Releasing

Uses Changesets for version management. All three packages are linked for versioning.

```bash
# Create a changeset (for documenting changes)
pnpm changeset

# Version packages based on changesets
pnpm version-packages

# Build and publish to npm
pnpm release
```

## Architecture

This is a pnpm monorepo using Turborepo for build orchestration.

### Packages

**Core Packages:**
- **@tour-kit/core** (`packages/core/`) - Framework-agnostic core logic, types, and utilities
- **@tour-kit/react** (`packages/react/`) - React components and hooks, depends on core
- **@tour-kit/hints** (`packages/hints/`) - Hint/beacon components, depends on core

**Extended Packages:**
- **@tour-kit/adoption** (`packages/adoption/`) - Feature adoption tracking and nudge system
- **@tour-kit/analytics** (`packages/analytics/`) - Plugin-based analytics integration
- **@tour-kit/announcements** (`packages/announcements/`) - Product announcements (modal, toast, banner, slideout, spotlight)
- **@tour-kit/checklists** (`packages/checklists/`) - Onboarding checklists with task dependencies
- **@tour-kit/media** (`packages/media/`) - Media embedding (YouTube, Vimeo, Loom, Wistia, GIF, Lottie)
- **@tour-kit/scheduling** (`packages/scheduling/`) - Time-based scheduling with timezone support
- **@tour-kit/surveys** (`packages/surveys/`) - In-app microsurveys (NPS, CSAT, CES) with fatigue prevention

### Build System

- **Turborepo** - Orchestrates builds with proper dependency ordering (`turbo.json`)
- **tsup** - Bundles each package, outputs ESM + CJS with TypeScript declarations
- **TypeScript** - Strict mode, ES2020 target, React JSX transform

### Package Dependencies

```
@tour-kit/react ─┐
                 ├──► @tour-kit/core
@tour-kit/hints ─┘
```

Both `react` and `hints` packages depend on `core`. Turbo handles build order automatically.

## Coding Rules

**You MUST follow the coding rules defined in `tour-kit/rules/`**. These rules ensure code quality, consistency, and maintainability.

### Rule Files

| File | Description |
|------|-------------|
| `typescript.md` | TypeScript strict mode, type patterns, generics |
| `react.md` | React component patterns, JSX conventions |
| `hooks.md` | Custom hook design and implementation |
| `components.md` | Component architecture and composition |
| `accessibility.md` | WCAG 2.1 AA compliance requirements |
| `testing.md` | Testing standards and coverage |
| `architecture.md` | Package structure and dependencies |
| `performance.md` | Bundle size budgets and optimization |

### Core Principles

1. **Headless First** - Logic in `@tour-kit/core`, components are thin wrappers
2. **Composition Over Configuration** - Small focused components that compose
3. **Type Safety** - Full TypeScript coverage with strict mode
4. **Accessibility First** - ARIA, focus management, keyboard navigation built-in
5. **Progressive Enhancement** - Works without JS, respects `prefers-reduced-motion`

### Quality Gates

- TypeScript strict mode enabled
- Test coverage > 80%
- Bundle sizes (gzipped): enforced by `tooling/bundle-check/check-dist-gzip.mjs`
  (the binding merge gate, raw `dist/index.js` gzip bytes — run `pnpm dist:size`).
  `size-limit` (root [`/.size-limit.json`](/.size-limit.json)) is a secondary
  smoke signal in a bundled-with-deps + brotli unit, run `pnpm bundlesize`.
  Per-package raw dist-gzip budgets:
  - core <20 KB (target <8 KB; tracked as audit B-1)
  - react <12 KB
  - hints <5 KB
  - analytics <4 KB (root; per-plugin <1.5 KB each)
  - adoption, checklists <10 KB
  - announcements, surveys, license <8 KB
  - media <6 KB
  - ai <5 KB (client), <8 KB (server)
  - scheduling <4 KB
- Lighthouse Accessibility: 100
- WCAG 2.1 AA compliant

## Execution Rules

- **Plan before acting:** For any task touching 3+ files or requiring debugging, state your plan in 2-3 bullet points BEFORE writing code. Wait for approval.
- **Structured debugging:** State your hypothesis before each fix attempt. Max 3 attempts before stepping back and asking for more context. Never shotgun-fix.
- **No rabbit holes:** Never spend more than 2 consecutive tool calls exploring/reading without producing output (code, a plan, or a concrete finding).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [domidex01/tour-kit](https://github.com/domidex01/tour-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
