---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# Agent Guide

This file provides guidance to AI coding agents when working with code in this repository.

## Cross-Agent Compatibility

- `AGENTS.md` is the primary entry point for Codex/OpenAI agents.
- `CLAUDE.md` exists for Claude-compatible tooling and should stay in sync with this file.
- In this repository, `AGENTS.md` is a symlink to `CLAUDE.md`, so editing `CLAUDE.md` updates both entry points.
- Skill files live under `.claude/skills/` for historical reasons, but they are plain Markdown playbooks that any agent can read and follow directly.
- If your environment does not support first-class "skills", open the relevant `SKILL.md` file and follow it manually.
- When another repo document says to "use a skill", interpret that as "read the matching `SKILL.md` file and follow its process" if your agent runtime does not have a native skill system.

## Self-Improvement

If you discover undocumented requirements, commands, or workflows during your work (e.g., a reviewer asks you to run something not covered here), update this file on the same PR. Keep this guide accurate and helpful for future agents.

## Project Overview

Codemod to transform styled-components to StyleX using jscodeshift.

## Commands

```bash
pnpm install
pnpm test:run    # Run tests once
pnpm check       # Run full validation (lint, typecheck, tests, knip, Storybook build, build, format)
```

## Rules

- src folder code should never depend on test-cases or test-case logic
- transformations should be safe and lossless, bail if we cannot preserve the semantics of the input
- adjacent sibling selectors (`+`) are not representable losslessly with current StyleX APIs; bail instead of approximating them as general siblings (`~`)
- run `pnpm check` to validate changes. It covers the same validation categories as `pnpm run ci`, so do not run both unless explicitly requested.
- when fixing bugs or addressing review comments, add test coverage to document the regression and prevent future breakage. **Prefer extending an existing test case** over creating a new one — only create a new test case when no existing case covers the same category/feature area
- when addressing PR review comments, resolve the addressed review threads after pushing the fix, then re-fetch thread state to verify they are resolved. Leave only unaddressed, ambiguous, or reviewer-confirmation-needed threads open.
- after creating a PR, make it ready for review (not draft), address actionable review comments, then poll for new PR review comments about every 90 seconds for up to 10 minutes. If new actionable comments appear, address them, push, resolve addressed threads, and repeat the same polling loop until no new review comments appear during a full polling window.
- before making any changes, explore the codebase to find ALL files that contain the pattern I'm about to describe. List every file, show the relevant code, and confirm you understand the full scope. Then propose a complete change plan covering every file.
- when `adapter.externalInterface` is `"auto"`, treat prepass as required: if prepass fails, throw (do not silently fall back); only function-based `externalInterface` may continue on prepass failure with a warning

## Code guidelines

- Prefer iteration & modularization over code duplication.
- **Unify and abstract**: Use existing primitives/helpers whenever possible. Extend or generalize shared utilities instead of adding parallel logic so the codebase uses the same primitives consistently.
- **Centralize common logic**: When adding new functionality, look for existing helper functions that can be extended rather than duplicating patterns. Key utilities like `literalToStaticValue` in `builtin-handlers.ts` handle AST node extraction and should be enhanced to support new node types rather than adding ad-hoc checks elsewhere.
- **keep all exports at the top of each file** (after imports), and keep **non-exported helpers further down**
- TypeScript: Avoid `any`; use proper types. (Some jscodeshift AST patterns are hard to type precisely—minimal, justified assertions are acceptable there.)
- Prefer type definitions; avoid type assertions (as, !) where feasible.
- Use descriptive names for variables & functions

## StyleX Constraints

StyleX does NOT support CSS shorthand properties. When transforming CSS to StyleX:

- `border` must expand to `borderWidth`, `borderStyle`, `borderColor`
- `margin`/`padding` must expand to directional properties (`marginTop`, etc.)
- `background` must map to `backgroundColor` or `backgroundImage`

**Key files for shorthand handling:**

- `src/internal/css-prop-mapping.ts` - `cssDeclarationToStylexDeclarations()` is the authoritative source for shorthand expansion
- `src/internal/lower-rules/borders.ts` - Handles interpolated border values
- Use `parseInterpolatedBorderStaticParts()` when parsing border values with dynamic expressions

When adding new CSS-to-StyleX transformations, always use these existing helpers rather than directly mapping CSS property names.

## StyleX Runtime Behavior


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skovhus/styled-components-to-stylex-codemod](https://github.com/skovhus/styled-components-to-stylex-codemod) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
